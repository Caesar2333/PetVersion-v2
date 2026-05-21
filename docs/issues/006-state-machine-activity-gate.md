# #6 — 状态机 + 活动追踪 + 交互门控

**Type:** AFK
**Blocked by:** #2（类型、roles）
**估时:** 5-7h

---

## What to build

行为核心三模块：ActivityTracker 监听原始浏览器事件发出语义 hook；InteractionGate 管理冷却+聚合特殊触发；PetStateMachine 纯同步函数根据 hook 和当前状态输出新 role。全部可独立单测。

## 涉及模块

### ActivityTracker

`src/pet-core/runtime/activityTracker.ts`

- 监听：keydown / input / click / scroll / wheel / mousemove / selectionchange / visibilitychange / focus / blur
- 节流：mousemove 每 100ms 最多一次；其他事件无节流但合并到同一 tick
- Settle timer：收到活跃事件后重置 3-5s 倒计时，到期发 `activitySettled`
- Inactivity timer：30-60s 无活动 || window blur || document.hidden → 发 `userInactive`
- 接口：
  - `start(listener: HookListener): void`
  - `stop(): void`
  - `getState(): { isActive, lastActivity, isInactive }`

### InteractionGate

`src/pet-core/runtime/interactionGate.ts`

- 维护 `cooldowns: Map<string, number>`（key: role name, value: timestamp）
- `allow(action, now): boolean` — 冷却期内返回 false
- `mark(action, now): void` — 记录本次触发时间
- 冷却时长：greet 5s, success 3s, special 3s, failed 2s
- `toSpecialTrigger(source): Hook` — 聚合 dblclick/contextmenu/longPress/hoverDwell 为 `specialTrigger`

### PetStateMachine

`src/pet-core/runtime/stateMachine.ts`

- 纯同步函数：`pickState(current: Role, hook: Hook, context: MachineContext): Role`
- 状态优先级（从高到低）：
  1. Dragging（move-left / move-right）
  2. Boundary Failed
  3. One-shot（greet / success / special / failed）
  4. Working
  5. Waiting
  6. Idle
- One-shot 锁：
  - 当前是 one-shot role 时，普通 hook（userActivity/activitySettled/userInactive/pointerEnter/clickPet/specialTrigger）不改变状态
  - 强交互 hook（dragStart/dragMoveLeft/dragMoveRight/boundaryFailed/viewportResize）可打断 one-shot
  - `animationEnd` hook → 按 action 的 `next` 规则（默认 idle）返回目标 role
- `MachineContext`：`{ viewportSize, petBounds, dragDelta }` 等运行时信息

## Acceptance criteria

### StateMachine (vitest)
- [ ] idle + userActivity → working
- [ ] working + activitySettled → idle
- [ ] idle + userInactive → waiting
- [ ] waiting + userActivity → working
- [ ] idle + pointerEnter → greet
- [ ] greet（播放中）+ userActivity → greet（不变，one-shot 锁）
- [ ] greet + animationEnd → idle
- [ ] greet（播放中）+ dragStart → move-left/move-right（drag 打断 one-shot）
- [ ] idle + clickPet → success
- [ ] idle + specialTrigger → special
- [ ] idle + viewportResize(clamped) → failed
- [ ] 同时多个 hook → 最高优先级胜出

### InteractionGate (vitest)
- [ ] greet 5s 冷却内 allow 返回 false
- [ ] 冷却过后 allow 返回 true
- [ ] mark 后时间戳更新
- [ ] toSpecialTrigger 正确聚合多种来源

### ActivityTracker (vitest + jsdom)
- [ ] keydown 触发 userActivity
- [ ] 3-5s 无事件触发 activitySettled
- [ ] 30-60s 无事件触发 userInactive
- [ ] document hidden 触发 userInactive
- [ ] 恢复可见 + 活动触发退出 inactive

## 禁止改动范围

- 不涉及 SpritePlayer 集成（那是 #14 的范围）
- 不涉及拖拽逻辑（那是 #7）
- 不涉及 chrome API 事件（用 jsdom mock）

## 验证方式

```bash
npx vitest run src/pet-core/runtime/stateMachine.test.ts
npx vitest run src/pet-core/runtime/interactionGate.test.ts
npx vitest run src/pet-core/runtime/activityTracker.test.ts
```
