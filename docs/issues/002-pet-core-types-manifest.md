# #2 — Pet Core：类型、角色 & Manifest 规范化

**Type:** AFK
**Blocked by:** None（纯 TypeScript，与 #1 并行）
**估时:** 4-6h

---

## What to build

定义全部领域类型 + 9 个标准 role + manifest 规范化流水线。全纯函数，零副作用，不依赖 chrome API。这是整个项目的类型基础，所有后续切片依赖此模块。

## 涉及模块

- `src/pet-core/types.ts`
  - `PetManifest`（id, displayName, spritesheet, actions, actionMap, preset?, extraActions?）
  - `ActionSpec`（row, frameCount, loop?, next?, durations?）
  - `AtlasSpec`（width, height, columns, rows, cellWidth, cellHeight）
  - `Role`（联合类型：9 个标准 role）
  - `NormalizedManifest`（所有字段必填，所有 fallback 已解析）
  - `Hook`, `MachineContext`, `PlaybackCompleteInfo`, `FrameInfo`

- `src/pet-core/roles.ts`
  - `ROLES` 常量数组：`idle | move-left | move-right | greet | working | waiting | success | failed | special`

- `src/pet-core/manifest/presets.ts`
  - `codex-9x9`：columns=9, rows=9, cellWidth=256, cellHeight=288
  - `hatch-8x9`：columns=8, rows=9, cellWidth=192, cellHeight=208

- `src/pet-core/manifest/normalizePetManifest.ts`
  - 接收 raw manifest + 图片 naturalWidth/naturalHeight
  - Spritesheet 补全优先级：完整字段 → preset 声明 → 自动检测匹配 → 失败返回 error
  - 为每个 action 补全 loop/next 默认值
  - 输出 `NormalizedManifest` 或 `NormalizeError`

- `src/pet-core/manifest/actionAliases.ts`
  - Alias 映射表：`waving/wave/hello → greet`, `running-left/walk-left → move-left`, `running-right/walk-right → move-right`, `jumping/happy → success`, `sad/fail/angry → failed`, `dance → special`

- `src/pet-core/manifest/actionResolver.ts`
  - 解析优先级：actionMap → 精确名匹配 → alias 表 → 未匹配
  - 输出 `{ actionMap: Record<Role, string>, extraActions: string[], unmappedRoles: Role[] }`
  - idle role 无映射时拒绝

- `src/pet-core/manifest/playbackRules.ts`
  - 默认规则：idle/move-left/move-right/working/waiting → loop:true；greet/success/failed/special → loop:false, next:"idle"
  - pet.json 显式字段覆写默认值

## Acceptance criteria

- [ ] 所有类型编译通过，无 `any`
- [ ] vitest 测试覆盖：
  - 完整 spritesheet 字段 → 直通不修改
  - 声明 preset → 缺失字段补全
  - 图片尺寸匹配已知 preset → 自动检测
  - 缺失 id → 拒绝
  - 未知 preset → 拒绝
  - 精确 action 名匹配 → 自动映射
  - alias 匹配 → 自动映射（waving → greet）
  - actionMap 显式指定 → 覆盖 alias
  - 未映射 role → 返回 unmappedRoles
  - idle 无映射 → 拒绝
  - 默认 loop/next → 正确应用
  - pet.json 覆写 loop/next → 覆写生效

## 禁止改动范围

- 不涉及 UI
- 不涉及 chrome API
- 不涉及存储

## 验证方式

```bash
npx vitest run src/pet-core/
```
