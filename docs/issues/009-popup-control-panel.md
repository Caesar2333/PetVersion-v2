# #9 — Popup：完整控制面板

**Type:** AFK
**Blocked by:** #1（构建）、#2（类型）、#3（存储）、#8（消息——开发阶段可用 mock）
**估时:** 5-7h

---

## What to build

Chrome 扩展 popup 面板，点击工具栏图标打开。React 19 + Tailwind + shadcn/ui + Motion + lucide-react。两个 Tab（Controls + Pets）完整交互。设计参考 `design/design-preview.html` 的 popup 部分和 `design/design.md` §3.1。

## 涉及模块

`src/app/popup/`

### 整体结构

```
┌─────────────────────────────────┐
│  Header bar                     │
│  [Pet icon] pet name  [Toggle]  │
├─────────────────────────────────┤
│  Tab bar: [Controls] [Pets]     │
├─────────────────────────────────┤
│  Tab content area (flex: 1)     │
├─────────────────────────────────┤
│  Footer: [Open Player] [Import] │
└─────────────────────────────────┘
```

### 子组件

- `PopupApp.tsx` — 根组件，管理 tab 状态、读取 storage、通过消息与 content script 通信
- `PetHeader.tsx` — 当前宠物 icon + displayName + preset 标签 + toggle switch
  - Toggle 动画：150ms ease-out，thumb 滑动 + track 颜色过渡
  - Disabled 时面板 60% opacity overlay
- `TabBar.tsx` — Controls / Pets 两个 tab，滑动下划线指示器（200ms ease-out）
- `ControlsTab.tsx`
  - Scale slider（0.5×–2×, step 0.1）— 右侧 monospace 数值实时显示
  - Speed slider（0.25×–2×, step 0.05）— 同上
  - Reset Position 按钮（secondary button）
- `PetsTab.tsx`
  - 3 列 pet card 网格
  - 每个卡片：72×72 缩略图区 + pet name + action count
  - 选中态：2px accent ring + scale(1.02)
  - 内置宠物网格 + 已导入宠物网格
- `PetCard.tsx` — 缩略图（可使用 spritesheet 第一帧或 emoji placeholder）、名称、action 数
- `PopupFooter.tsx` — "Open Player"（secondary） + "Import Pet"（primary）按钮

### 状态处理

- **Loading:** Pets tab 显示 3 个骨架卡片（pulsing grey rectangle）
- **Empty:** "No pets installed" 居中提示 + "Import your first pet" CTA
- **Error:** 红色 banner "Failed to load pets" + retry 链接
- **Disabled:** 面板 60% opacity 遮罩，toggle 显示 off

### 通信

- 读取：popup 打开时从 chrome.storage.local 加载 pet list + settings
- 写入：slider 变化 → `PET_UPDATE_GLOBAL_STATE` → content script 实时响应
- 导航：`chrome.tabs.create({ url: 'player.html' })` / `chrome.tabs.create({ url: 'import.html' })`

## Acceptance criteria

- [ ] 点击扩展图标 → popup 打开，显示当前宠物名 + toggle（on）
- [ ] Toggle off → 页面上宠物消失；toggle on → 宠物重新出现
- [ ] Scale slider 拖动 → 页面上宠物实时缩放，数值显示正确
- [ ] Speed slider 拖动 → 宠物播放速度实时变化
- [ ] Reset Position → 宠物回到右下角默认位置
- [ ] Controls tab ↔ Pets tab 切换流畅，下划线滑动
- [ ] Pets tab 显示宠物网格，点击切换宠物 → 页面宠物实时切换
- [ ] 选中宠物卡片有 accent ring + scale 效果
- [ ] "Open Player" → 新标签页打开 player.html
- [ ] "Import Pet" → 新标签页打开 import.html
- [ ] 暗色模式（system preference）正常工作
- [ ] 键盘 Tab 导航顺序正确

## 禁止改动范围

- 不做 player/import 页面内容（那是 #10/#11）
- 不修改 content script 行为逻辑（那是 #5/#6/#7）

## 验证方式

`vite build` → Chrome 加载扩展 → 点击工具栏图标 → popup 打开。逐一操作每个控件，确认页面宠物实时响应。切换 tab → 确认 UI 正确。点击底部按钮 → 确认跳到正确页面。
