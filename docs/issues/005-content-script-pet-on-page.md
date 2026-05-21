# #5 — Content Script：宠物出现在网页上

**Type:** AFK
**Blocked by:** #1（构建产物）、#2（类型）、#4（SpritePlayer）
**估时:** 3-4h

---

## What to build

Content script 注入到任意 HTTP/HTTPS 页面，创建 Shadow DOM overlay，实例化 SpritePlayer 加载宠物并播放 idle 动画。这是宠物在网页上"可见"的最小闭环。

## 涉及模块

- `src/content/content.ts`
  - 注入入口，`document_idle` 时机
  - 仅 top window（`window.top === window.self`）
  - 排除 URL 正则：`chrome://`、`edge://`、`about:`、`chrome-extension://`、`chrome.google.com/webstore`、`chromewebstore.google.com`

- `src/content/injectPet.ts`
  - 创建 Shadow DOM host div → `document.body.appendChild`
  - 挂载 Shadow Root（mode: 'closed'）
  - 注入 `petOverlay.css`（fixed 定位、z-index: 2147483647、pointer-events: none on container, auto on pet shell）
  - 创建 `shell` 和 `sprite` 两个 DOM 元素
  - 从 storage 读取当前选中宠物 manifest + IndexedDB Blob
  - `URL.createObjectURL(blob)` → 传给 SpritePlayer
  - 实例化 SpritePlayer → `playRole('idle')`
  - 默认位置：右下角，margin 24px
  - 页面卸载时 `destroy()` + `URL.revokeObjectURL`

- `src/content/petOverlay.css`
  - Shadow DOM 内完全隔离的样式
  - Host container: `position: fixed; inset: 0; pointer-events: none; z-index: 2147483647`
  - Pet shell: `position: absolute; pointer-events: auto; user-select: none; cursor: grab`
  - Pet sprite: `width: 100%; height: 100%; background-repeat: no-repeat`

## Acceptance criteria

- [ ] `vite build` → Chrome 加载扩展 → 访问 `baidu.com` → 宠物出现在右下角
- [ ] 宠物播放 idle 循环动画
- [ ] 访问 `chrome://extensions` → 宠物不出现
- [ ] 含 iframe 的页面 → 宠物仅在顶层窗口出现（不在 iframe 内重复）
- [ ] 宠物不阻挡页面文本选择（非宠物区域）
- [ ] 刷新页面 → 宠物重新出现
- [ ] 关闭标签页 → 无 JS 错误

## 禁止改动范围

- 不做状态机行为切换（idle 硬编码即可，那是 #6 的范围）
- 不做拖拽交互（那是 #7 的范围）
- 不做 popup 通信（那是 #8 的范围）
- 不做 scale/speed 控制（那是后续范围）

## 验证方式

加载扩展到 Chrome → 访问 3 个不同网站（如 baidu.com, github.com, zhihu.com）→ 确认宠物均出现 → 访问 chrome://extensions → 确认宠物不出现。
