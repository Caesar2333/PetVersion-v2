# #8 — Background Service Worker + 消息通信

**Type:** AFK
**Blocked by:** #2（类型）、#3（存储）
**估时:** 3-4h

---

## What to build

Background Service Worker 作为消息中枢：路由 popup ↔ content script 通信，读写全局状态，广播变更。消息类型常量和类型安全 payload 定义。

## 涉及模块

- `src/shared/messages.ts`
  - 消息类型常量：
    - `PET_GET_GLOBAL_STATE` — content script 请求当前全局状态
    - `PET_UPDATE_GLOBAL_STATE` — content/popup 更新全局状态 patch
    - `PET_GLOBAL_STATE_UPDATED` — background 广播状态变更
    - `PET_GET_STATUS` — popup/player 查询当前 pet 状态
    - `PET_STATUS_RESPONSE` — 返回当前状态快照
    - `PET_RESET_POSITION` — 重置位置到默认
  - 每个消息的 Payload 和 Response 类型

- `src/background/service-worker.ts`
  - `chrome.runtime.onMessage` 监听
  - 消息路由：
    - `PET_GET_GLOBAL_STATE` → 从 chrome.storage.local 读取 settings + selectedPetId → 返回
    - `PET_UPDATE_GLOBAL_STATE` → 合并 patch 到 chrome.storage.local → 广播 `PET_GLOBAL_STATE_UPDATED` 到所有 tabs
    - `PET_GET_STATUS` → 返回当前状态快照
    - `PET_RESET_POSITION` → 清除存储的位置 → 广播重置
  - `chrome.runtime.onInstalled` → 初始化默认 settings
  - 错误处理：catch 所有 handler，记录 console.error

- Content script 侧：
  - 启动时发 `PET_GET_GLOBAL_STATE` 获取初始状态
  - 监听 `PET_GLOBAL_STATE_UPDATED` → 更新本地 SpritePlayer（scale/speed/pet切换）
  - 拖拽结束发 `PET_UPDATE_GLOBAL_STATE` 保存位置

- Popup 侧（预备接口，UI 在 #9）：
  - 发 `PET_GET_STATUS` 获取当前状态展示
  - 发 `PET_UPDATE_GLOBAL_STATE` 修改 settings
  - `chrome.tabs.create` 打开 player/import 页面

## Acceptance criteria

- [ ] Service worker 启动成功（Chrome DevTools → Service Worker console 无错误）
- [ ] Content script 启动 → 发 `PET_GET_GLOBAL_STATE` → 收到 settings 响应
- [ ] Popup 发 `PET_UPDATE_GLOBAL_STATE` → content script 收到 `PET_GLOBAL_STATE_UPDATED` 广播
- [ ] `PET_RESET_POSITION` → content script 宠物回到默认位置
- [ ] chrome.storage.local 读写正确
- [ ] 所有消息 payload 类型安全

## 禁止改动范围

- 不实现 popup UI（那是 #9）
- 不实现 player/import 页面通信细节（那是 #10/#11）

## 验证方式

加载扩展 → Chrome DevTools → Application → Service Workers → 确认 SW 激活。

打开任意网页 → Console → 确认 content script 发送 `PET_GET_GLOBAL_STATE` 并收到响应。

从 popup（临时 console）发送 `PET_UPDATE_GLOBAL_STATE` → 确认 content script 收到广播。
