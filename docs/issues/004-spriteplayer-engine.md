# #4 — SpritePlayer 引擎

**Type:** AFK
**Blocked by:** #2（AtlasSpec、ActionSpec、Role 类型）
**估时:** 4-6h

---

## What to build

按 PRD §4.9 实现 `SpritePlayer` 类。通过 `requestAnimationFrame` 驱动 CSS `background-position` 逐帧播放 spritesheet。帧时长优先取 action 的 `durations[]`，fallback 到全局 `frameDuration`（150ms），乘以 `playbackRate` 缩放。一次性动画结束后回调 `onComplete`。

## 涉及模块

- `src/pet-core/runtime/spritePlayer.ts`
  - `constructor({ shell, sprite, spritesheetUrl, atlas, actions, actionMap, initialScale, playbackRate, frameDuration, onComplete, onFrame })`
  - `playRole(role)` — 主接口：状态机使用。通过 actionMap 查 actionId → playAction
  - `playAction(actionId)` — 调试接口：player 页面使用
  - `pause()` / `resume()` / `stop()`
  - `setScale(scale)` — CSS transform scale
  - `setPlaybackRate(rate)` — 0.25x–2x 范围
  - `getSnapshot(): FrameInfo` — 当前帧、actionId、role 等
  - `destroy()` — 清理 raf、移除 DOM

- 核心渲染逻辑：
  - CSS `background-image` + `background-position` + `background-size`
  - `background-position-x = -(frameIndex % columns) * cellWidth * scale`
  - `background-position-y = -(row) * cellHeight * scale`
  - `background-size = atlasWidth * scale + 'px ' + atlasHeight * scale + 'px'`
  - 帧时长：action.durations?.[frameIndex] ?? frameDuration，effective = base / playbackRate
  - One-shot：播放完最后一帧 → `onComplete({ role, actionId, next })`；loop：回到第一帧
  - `initialScale`/`scale` 变化 → 重设 shell 宽高（cellWidth * scale, cellHeight * scale）
  - `playbackRate` 变化 → 立即生效，不重置当前帧
  - 切换 role 时：如果是同一 action → 不做任何事；不同 action → 重置到新 action 第 0 帧
  - 暂停/恢复 → 帧进度保持
  - `stop()` → 清空 background-image，不发 onComplete

## Acceptance criteria

- [ ] 构建独立测试 HTML：传入测试 spritesheet URL + atlas spec → `playRole('idle')` → 9 帧循环播放
- [ ] `playbackRate = 2` → 帧速翻倍（视觉可感知）
- [ ] `playbackRate = 0.5` → 帧速减半
- [ ] One-shot action（greet）→ 播放完所有帧 → fire `onComplete({ role:'greet', actionId:'greet', next:'idle' })`
- [ ] Loop action（idle）→ 不 fire onComplete，持续循环
- [ ] `setScale(1.5)` → shell 元素变大，background-size 相应放大
- [ ] `pause()` → 帧冻结；`resume()` → 从冻结帧继续
- [ ] `stop()` → 清空背景
- [ ] `getSnapshot()` 返回当前 frameIndex、actionId、role
- [ ] 性能：60fps 稳定，`requestAnimationFrame` 内无 layout thrashing

## 禁止改动范围

- 不涉及状态机逻辑
- 不涉及 chrome API
- 不涉及存储

## 验证方式

`src/pet-core/runtime/__tests__/` 下建测试 HTML 或用 vitest + jsdom 模拟 DOM 操作验证。
