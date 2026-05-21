# #13 — 测试套件

**Type:** AFK
**Blocked by:** #2、#6、#7（测试这些切片中实现的纯逻辑模块）
**估时:** 3-4h

---

## What to build

vitest 测试套件，覆盖所有纯逻辑模块。测试配置 + 5 个测试文件。持续在相关模块实现后补充测试。

## 涉及模块

### 测试配置

- `vitest.config.ts` — jsdom environment、include 路径、coverage 配置（v8）
- `package.json` scripts：`"test": "vitest run"`, `"test:watch": "vitest"`

### 测试文件

`src/pet-core/manifest/__tests__/normalizePetManifest.test.ts`
- 完整 spritesheet 字段 → pass-through 不修改
- 声明 preset "codex-9x9" → 缺失字段正确补全
- 声明 preset "hatch-8x9" → 缺失字段正确补全
- 图片尺寸 2304×2592 → 自动检测 codex-9x9
- 图片尺寸 1536×1872 → 自动检测 hatch-8x9
- 未知图片尺寸 + 无 preset → 返回 error
- pet.json 缺 id → 拒绝
- 未知 preset 字符串 → 拒绝
- actions 缺失 loop/next → 补全默认值
- actions 显式 loop:false, next:"greet" → 保留不覆盖
- extra actions 不在 actionMap 中 → 归入 extraActions

`src/pet-core/manifest/__tests__/actionResolver.test.ts`
- 9 个 action 名正好是 role 名 → 全自动匹配
- "waving" → alias 匹配到 greet
- "running-left" → alias 匹配到 move-left
- "running-right" → alias 匹配到 move-right
- "happy" → alias 匹配到 success
- "angry" → alias 匹配到 failed
- "dance" → alias 匹配到 special
- "wave" → alias 匹配到 greet
- actionMap 显式指定 → 覆盖 alias 结果
- 未知 action → 不匹配任何 role，进入 extraActions
- 多个 action 映射同一 role → 选第一个匹配的
- idle 无映射 → reject

`src/pet-core/runtime/__tests__/stateMachine.test.ts`
- idle + userActivity → working
- working + activitySettled → idle
- idle + userInactive → waiting
- waiting + userActivity → working
- idle + pointerEnter → greet
- greet(播放中) + userActivity → 保持 greet（one-shot 锁）
- greet + animationEnd → idle
- greet(播放中) + dragStart → move-left/move-right（drag 打断）
- idle + clickPet → success
- idle + specialTrigger → special
- idle + viewportResize(clamped) → failed
- drag + userActivity → 保持 drag（drag 最高优先级）
- 同时多个 hook → 取最高优先级
- animationEnd with next:"greet" → greet（非默认 idle）

`src/pet-core/runtime/__tests__/interactionGate.test.ts`
- greet allow → 冷却期内 false，过期后 true
- mark 后 allow 立即变 false
- success/special/failed 各冷却时长正确
- toSpecialTrigger 聚合 dblclick/contextmenu/longPress/hoverDwell → specialTrigger
- 不同 action 冷却互不干扰

`src/pet-core/runtime/__tests__/bounds.test.ts`
- 正常位置在视口内 → 不 clamp
- x < 0 → clamp 到 0
- y < 0 → clamp 到 0
- x > viewportWidth - petWidth → clamp 到边界
- y > viewportHeight - petHeight → clamp 到边界
- 不同 scale 下 petWidth/petHeight 计算正确
- 小视口（如 200×200）→ 宠物居中（或尽可能接近）
- equal to edge → 不算越界

## Acceptance criteria

- [ ] `npx vitest run` 全部通过
- [ ] 每个模块的核心路径和边界情况有覆盖
- [ ] 纯逻辑模块覆盖率 ≥80%
- [ ] 测试可独立运行，不依赖 chrome API

## 禁止改动范围

- 不测 DOM 交互（content script 注入、拖拽）
- 不测 chrome.storage / IndexedDB（需扩展环境）
- 不测 UI 组件渲染

## 验证方式

```bash
npx vitest run        # CI 模式
npx vitest            # watch 模式
npx vitest run --coverage  # 覆盖率报告
```
