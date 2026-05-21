# #10 — Player 页面：完整调试控制台

**Type:** AFK
**Blocked by:** #1（构建）、#2（类型）、#3（存储）、#4（SpritePlayer）
**估时:** 6-8h

---

## What to build

独立的 player.html 全页调试控制台。双栏布局：左侧宠物视口 + 右侧可滚动调试面板。可逐一触发 9 个 role、手动发送 hook、查看 spritesheet 参数和 actionMap。设计参考 `design/design-preview.html` 的 player 部分和 `design/design.md` §3.2。

## 涉及模块

`src/app/player/`

### 整体布局

```
┌──────────────────────────────────────────────────┐
│  Top bar: [← Back] pet▾ [Scale] [Speed] Bg▾ [☀] │
├──────────────────────┬───────────────────────────┤
│  PET VIEWPORT (60%)  │  DEBUG PANELS (40%)       │
│  ┌──────────────┐   │                           │
│  │ [sprite]     │   │  State Machine status      │
│  │              │   │  Role Triggers (9 btns)    │
│  │              │   │  Hook Triggers             │
│  └──────────────┘   │  Spritesheet Info          │
│  bg: checkerboard   │  Current Action            │
│                     │  Action Map table           │
└──────────────────────┴───────────────────────────┘
```

### 子组件

- `PlayerApp.tsx` — 根组件，管理 pet 选择、当前 role、last hook 状态
- `PlayerTopBar.tsx`
  - Back 按钮（ghost）
  - Pet 下拉选择器（Select 组件）
  - Scale badge + Speed badge（monospace）
  - 背景切换下拉（Checkerboard / Transparent / Light / Dark）
  - 暗色模式 toggle 按钮
- `PetViewport.tsx`
  - 320×320 stage，inner shadow
  - SpritePlayer 实例渲染
  - 4 种背景模式切换
  - "Checkerboard" CSS pattern（12px 网格）
  - Role 触发时短暂 glow（200ms accent border）
- `StateMachinePanel.tsx`
  - Current role badge（accent fill）
  - Last hook 显示（monospace）
  - 实时更新
- `RoleTriggersPanel.tsx`
  - 3×3 grid，9 个 role 按钮
  - 当前 active role 按钮高亮（accent bg + white text）
  - 点击 → `playRole(role)` → stage glow → 1s 后（oneshot）或持续（loop）高亮
- `HookTriggersPanel.tsx`
  - 紧凑按钮，按来源分组：Activity / Pointer / Drag / Window
  - userActivity, activitySettled, userInactive
  - pointerEnter, clickPet, specialTrigger
  - dragStart, dragMoveLeft, dragMoveRight, dragEnd
  - windowBlur, windowFocus, viewportResize
- `SpritesheetInfoPanel.tsx`
  - 3×2 stat grid：Columns, Rows, Cell W, Cell H, Atlas W, Atlas H
  - Monospace 数值
- `CurrentActionPanel.tsx`
  - role → action 映射显示
  - Row / Frames 数
  - Loop / Next 规则
- `ActionMapPanel.tsx`
  - 紧凑表格：Role | Action | Status
  - Status badges：✓ auto（green）、↻ alias（amber）、✗ unmapped（red）、✎ manual（blue）

### 通信

- 加载时从 chrome.storage.local 读取 pet list
- 选中 pet → 从 IndexedDB 加载 spritesheet Blob → `URL.createObjectURL` → 实例化 SpritePlayer
- 切换 pet → 销毁旧 SpritePlayer → revoke URL → 加载新 pet
- Role trigger → `spritePlayer.playRole(role)`
- Hook trigger → 调用状态机 `pickState()` → 结果更新 UI 显示
- Scale/Speed → `spritePlayer.setScale/setPlaybackRate`

## Acceptance criteria

- [ ] 打开 player.html → 顶部栏显示宠物下拉选择器
- [ ] 选择宠物 → 视口显示精灵动画
- [ ] 逐一触发 9 个 role 按钮 → 正确动画播放 + 按钮高亮
- [ ] Oneshot role 播放完自动恢复；loop role 持续播放
- [ ] 点击 hook 按钮 → last hook 显示更新
- [ ] 背景切换：checkerboard → transparent → light → dark → 循环
- [ ] Scale/Speed 调整 → 精灵实时变化
- [ ] Spritesheet Info 面板数据与 pet.json 一致
- [ ] Current Action 面板实时反映当前播放状态
- [ ] ActionMap 表格显示正确映射和状态徽章
- [ ] 暗色模式 toggle 正常工作
- [ ] 响应式：窗口缩窄时面板切换到堆叠布局

## 禁止改动范围

- 不修改 SpritePlayer 核心逻辑（那是 #4）
- 不修改状态机（那是 #6）

## 验证方式

`vite build` → Chrome 加载扩展 → popup → "Open Player" → 新标签页打开。逐一测试每个控件和面板，确认数据准确、动画正确。
