# #7 — 拖拽控制器 & 边界计算

**Type:** AFK
**Blocked by:** #2（cellWidth/cellHeight 从 manifest 动态读取）
**估时:** 3-4h

---

## What to build

宠物在页面上可拖拽移动。DragController 处理 pointer events 并发出语义 hook。Bounds 模块根据当前宠物 manifest 动态计算边界并 clamp 位置。越界修正时触发 `viewportResize` hook 供状态机触发 `failed` 动画。

## 涉及模块

### DragController

`src/pet-core/runtime/dragController.ts`

- 绑定到 pet shell 元素：
  - `pointerdown` → 记录起始位置，`setPointerCapture`，发 `dragStart`
  - `pointermove` → 计算 dx/dy，更新位置（CSS translate），dx < 0 发 `dragMoveLeft`，dx > 0 发 `dragMoveRight`
  - `pointerup` → 释放 capture，发 `dragEnd`，通知位置保存
- 拖拽中：`user-select: none`，`cursor: grabbing`
- 接口：
  - `attach(shell: HTMLElement, onHook: HookListener, onPositionChange: PositionCallback): void`
  - `detach(): void`

### Bounds

`src/pet-core/runtime/bounds.ts`

- `clampPosition(x, y, viewportWidth, viewportHeight, cellWidth, cellHeight, scale): ClampResult`
  - 宠物实际像素尺寸 = `cellWidth * scale` × `cellHeight * scale`
  - 边界范围：`0 ≤ x ≤ viewportWidth - petWidth`, `0 ≤ y ≤ viewportHeight - petHeight`
  - 默认 margin 24px 体现在初始位置，不在 clamp 逻辑中
- `getDefaultPosition(viewport, cellWidth, cellHeight, scale): {x, y}`
  - 右下角，offset 24px
- `isOutOfBounds(x, y, viewport, cellWidth, cellHeight, scale): boolean`

### 位置持久化

- Content script 中：
  - DragController `onPositionChange` → 写 `chrome.storage.local`（或通过 SW）
  - 宠物初始化时读取上次保存的位置
  - 无保存位置时用 `getDefaultPosition`
- Resize 时重新 clamp 当前位置，若发生修正 → 通知状态机

## Acceptance criteria

- [ ] 拖拽宠物到页面任意位置 → 宠物跟随指针
- [ ] 拖拽中页面文本不可选中
- [ ] 释放鼠标 → 宠物停在释放位置，位置保存
- [ ] 拖到左/上边界外 → 宠物被 clamp 在边界内
- [ ] 拖到右/下边界外 → 宠物被 clamp
- [ ] 缩小浏览器窗口至宠物越界 → 宠物自动修正到可见区域
- [ ] 刷新页面 → 宠物在上次拖拽的位置出现
- [ ] 首次加载（无保存位置）→ 宠物在右下角

## 禁止改动范围

- 不做状态机集成（保持 hook 回调即可，状态机在 #6）
- 不做自动行走/寻路（Non-Goal）
- 不做动画跟随缓动（直接 translate，不引入 spring）

## 验证方式

加载扩展 → 访问任意网页 → 拖拽宠物到各个角落 → 确认跟随、边界限制、位置保存。缩小窗口 → 确认自动修正。
