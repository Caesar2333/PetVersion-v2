# PRD: petExtension-v2 — Chrome 浏览器宠物插件

## 1. Problem Statement

用户希望在浏览网页时，有一个桌面宠物悬浮在页面上方，伴随用户的浏览行为展示不同动画。现有的参考项目 PetExtension 存在以下问题：

- **spritesheet 规格硬编码**：播放器写死了 8×9 grid、192×208 cell，无法适配新的 9×9/256×288 格式或其他尺寸的宠物资源。
- **状态名硬编码**：`HATCH_PET_SPEC` 和状态机锁死了 9 个固定 action 名（`running-left`、`waving` 等）。导入自定义宠物时，如果 action 名不匹配就无法工作。
- **pet.json 缺乏自描述能力**：没有 spritesheet 尺寸字段、没有 actionMap、没有 loop/next 播放规则。一切依赖代码里的常量。
- **导入流程脆弱**：旧格式宠物 spritesheet 转 base64 dataUrl 塞进 `chrome.storage.local`，多宠物场景下容易触碰 10MB 上限。
- **UI 简陋**：popup/player/import 页面使用手工 CSS，无设计系统，视觉效果和交互体验有限。

v2 需要解决上述所有问题，同时保持参考项目的核心架构优势（Chrome Extension MV3、Shadow DOM overlay、事件驱动状态机、拖拽交互）。

## 2. Solution

一个 Chrome Extension MV3 浏览器宠物插件，在任意网页上渲染一个可拖拽的动画宠物。宠物行为由用户页面活动驱动（鼠标、键盘、滚轮、窗口焦点等），通过状态机自动切换动画。

核心改进：

- **动态 spritesheet 解析**：播放器和边界计算完全从 pet.json 的 `spritesheet` 字段读取，支持任意尺寸。
- **Role-Based Action Mapping**：状态机只认 9 个标准语义 role，通过 `actionMap` 映射到宠物的真实 action 名，实现宠物资源与状态机解耦。
- **Preset 兼容系统**：内置 `codex-9x9` (256×288) 和 `hatch-8x9` (192×208) 两个 preset，自动补全旧格式宠物缺少的 spritesheet 参数。
- **分层存储**：chrome.storage.local 存元数据，IndexedDB 存 spritesheet Blob，解除单宠物存储上限。
- **高质量 UI**：popup/player/import 三个独立页面使用 React + Tailwind CSS v3 + shadcn/ui + lucide-react + Motion，统一视觉语言。

技术栈：Vite 7 + React 19 + TypeScript + Tailwind CSS v3 + shadcn/ui + Motion for React + lucide-react

打包方式：手动 `rollupOptions.input` 多入口（沿用参考项目已验证方案），不引入 CRXJS。

## 3. Navigation Model（三个独立页面）
      31
      32 +Chrome Extension 标准架构：**popup + 两个独立 HTML 页面**。
      33 +
      34 +```
      35 +┌─────────────────────────────────────────────┐
      36 +│  Chrome 工具栏                               │
      37 +│  [📌 petExtension 图标]  ← 用户点击           │
      38 +└─────────────────────────────────────────────┘
      39 +                    │
      40 +                    ▼
      41 +         ┌──────────────────┐
      42 +         │   popup.html     │  ← 弹出小面板 (~400×500px)
      43 +         │   控制面板         │     点击扩展图标打开
      44 +         │                  │
      45 +         │  [Open Player] ──┼──→  chrome.tabs.create → player.html（独立全页）
      46 +         │  [Import Pet]  ──┼──→  chrome.tabs.create → import.html（独立全页）
      47 +         └──────────────────┘
      48 +```
      49 +
      50 +- **popup.html**：Chrome 扩展弹出面板。用户点击工具栏图标 → 打开此小窗口 → 进行日常控制（开关、缩放、切换宠物）→ 关闭自动消失。
      51 +- **player.html**：独立全页。通过 popup 内按钮 `chrome.tabs.create` 打开新标签页。用于宠物动画调试和状态机验证。
      52 +- **import.html**：独立全页。通过 popup 内按钮 `chrome.tabs.create` 打开新标签页。用于导入自定义宠物。
      53 +
      54 +三个页面都使用 React + Tailwind + shadcn/ui。popup 和 player/import 之间不共享同一个窗口 — 后者是完整的独立浏览器标签页。

## 3. User Stories

### US-1: 日常使用 — 宠物在网页上自然交互

**As** 一个浏览器用户
**I want** 在浏览任意网页时看到宠物在页面角落活动
**So that** 浏览体验更有趣、不枯燥

Acceptance:
- 打开任意 HTTP/HTTPS 页面，宠物自动出现在右下角
- 宠物默认播 idle 动画（循环）
- 鼠标滑入宠物区域 → 宠物播 greet 动画（一次性，结束后回 idle）
- 单击宠物 → 宠物播 success 动画（一次性）
- 双击/右键/长按/长悬停宠物 → 宠物播 special 动画（一次性）
- 拖拽宠物 → 宠物播 move-left 或 move-right 动画，跟随鼠标
- 用户在页面上持续操作（键盘输入、点击、滚轮、选择文字等）→ 宠物播 working 动画（循环）
- 用户停止操作 3-5 秒后 → 宠物回到 idle
- 用户长时间无操作 30-60 秒，或浏览器标签页切到后台/隐藏 → 宠物播 waiting 动画（循环）
- 页面恢复焦点/用户重新操作 → 宠物退出 waiting，进入 working
- 窗口 resize 导致宠物越界被自动修正 → 宠物播 failed 动画（一次性）
- 所有交互有冷却时间，避免状态疯狂闪烁

### US-2: 弹出面板 — 日常控制

**As** 一个用户
**I want** 点击扩展图标打开一个控制面板
**So that** 可以快速切换宠物、调整大小、开关宠物显示

layout参考下面这个

```
  31
  32 +Chrome Extension 标准架构：**popup + 两个独立 HTML 页面**。
  33 +
  34 +```
  35 +┌─────────────────────────────────────────────┐
  36 +│  Chrome 工具栏                               │
  37 +│  [📌 petExtension 图标]  ← 用户点击           │
  38 +└─────────────────────────────────────────────┘
  39 +                    │
  40 +                    ▼
  41 +         ┌──────────────────┐
  42 +         │   popup.html     │  ← 弹出小面板 (~400×500px)
  43 +         │   控制面板         │     点击扩展图标打开
  44 +         │                  │
  45 +         │  [Open Player] ──┼──→  chrome.tabs.create → player.html（独立全页）
  46 +         │  [Import Pet]  ──┼──→  chrome.tabs.create → import.html（独立全页）
  47 +         └──────────────────┘
  48 +```
  49 +
  50 +- **popup.html**：Chrome 扩展弹出面板。用户点击工具栏图标 → 打开此小窗口 → 进行日常控制（开关、缩放、切换宠物）→ 关闭自动消失。
  51 +- **player.html**：独立全页。通过 popup 内按钮 `chrome.tabs.create` 打开新标签页。用于宠物动画调试和状态机验证。
  52 +- **import.html**：独立全页。通过 popup 内按钮 `chrome.tabs.create` 打开新标签页。用于导入自定义宠物。
  53 +
  54 +三个页面都使用 React + Tailwind + shadcn/ui。popup 和 player/import 之间不共享同一个窗口 — 后者是完整的独立浏览器标签页。
```



Acceptance:
- 点击浏览器工具栏的扩展图标 → 打开 popup 面板
- 面板顶部显示当前宠物名称 + 启用/禁用 toggle
- 有滑块来控制scale，也就是宠物的大小
- 有滑块来控制speed，也就是播放的速度
- Pets tab: 内置宠物网格 + 已导入宠物网格（如有），点击切换
- Pets tab: 底部"打开播放器"和"导入宠物"按钮

### US-3: 动画播放器 — 调试与预览

**As** 一个用户/开发者
**I want** 在一个独立页面里预览宠物动画和调试状态机
**So that** 可以验证 spritesheet 解析是否正确、action 映射是否正确、动画表现是否符合预期

Acceptance:
- 独立的 player.html 页面
- 可以选择已安装的任意宠物
- 可以逐一触发 9 个标准 role（idle/move-left/move-right/greet/working/waiting/success/failed/special）
- 可以手动触发状态机 hook（userActivity/activitySettled/userInactive/pointerEnter/clickPet/specialTrigger/dragStart/dragMove/dragEnd）
- 可以调整 scale 和 playbackRate
- 显示当前 spritesheet 参数（columns/rows/cellWidth/cellHeight/atlasWidth/atlasHeight）
- 显示当前播放的 action 详情（row/frameCount/loop/next）
- 显示 actionMap（role → action name）
- 支持背景切换（transparent/checkerboard/light/dark）以便对比动画边缘

### US-4: 导入宠物 — 扩展宠物库

**As** 一个用户
**I want** 导入自己的宠物资源（pet.json + spritesheet 图片）
**So that** 可以使用自定义宠物，而不限于内置宠物

Acceptance:
- 独立的 import.html 页面
- 上传 pet.json 文件
- 上传 spritesheet 图片（webp/png）
- 系统自动读取图片 naturalWidth/naturalHeight
- 系统自动补全 spritesheet 参数（优先用 pet.json 已有字段 → preset 声明 → 自动检测匹配 → 手动选择 preset）
- 系统自动解析 actions 并尝试生成 actionMap（精确名匹配 → alias 表 → 未匹配的提示手动映射）
- 展示检测结果：spritesheet 参数、actions 列表、actionMap 预览
- 用户可修正 9 个标准 role 的映射（下拉选择对应的 action）
- 未映射的 action 列为 extraActions，保留但不进入状态机
- 用户确认后，生成 normalized manifest 存入 chrome.storage.local
- spritesheet Blob 存入 IndexedDB
- 导入完成后该宠物出现在 popup 的宠物列表中

## 4. Implementation Decisions

本节记录所有已确认的实现级决策，按模块组织。

### 4.1 架构与打包

| 决策 | 结论 |
|------|------|
| 框架 | Vite 7 + React 19 + TypeScript（不用 Next.js） |
| 打包 | 手动 `rollupOptions.input` 多入口，沿用参考项目方案 |
| 入口点 | 5 个：`content.ts`、`service-worker.ts`、`popup.html`、`player.html`、`import.html` |
| 开发模式 | 双模式：`vite dev`（UI HMR + mock extensionApi）+ `vite build`（完整扩展验证） |
| UI 样式 | Tailwind CSS v3 + shadcn/ui，禁止 per-page 独立 style.css |
| 图标 | lucide-react |
| 动画库 | Motion for React（用于 UI 过渡动效；宠物 spritesheet 动画由 SpritePlayer 独立渲染） |

### 4.2 目录结构

```
src/
  app/
    popup/          main.tsx + PopupApp.tsx + components/
    player/         main.tsx + PlayerApp.tsx + components/
    import/         main.tsx + ImportApp.tsx + components/
  background/
    service-worker.ts
  content/
    content.ts
    injectPet.ts
    petOverlay.css
  pet-core/
    types.ts
    roles.ts
    manifest/
      normalizePetManifest.ts
      presets.ts
      actionAliases.ts
      actionResolver.ts
      playbackRules.ts
    runtime/
      spritePlayer.ts
      stateMachine.ts
      activityTracker.ts
      interactionGate.ts
      dragController.ts
      bounds.ts
    repository/
      petCatalog.ts
      petRepository.ts
      loadPet.ts
  storage/
    extensionApi.ts
    chromeStorage.ts
    indexedDbAssets.ts
  shared/
    messages.ts
    constants.ts
  components/
    ui/             shadcn/ui 组件
  styles/
    globals.css     Tailwind + shadcn CSS variables
```

### 4.3 Manifest V3 权限

```json
{
  "manifest_version": 3,
  "permissions": ["storage", "unlimitedStorage", "activeTab"],
  "host_permissions": ["http://*/*", "https://*/*"],
  "web_accessible_resources": [{
    "resources": [
      "pets/catalog.json",
      "pets/*/pet.json",
      "pets/*/spritesheet.webp",
      "pets/*/spritesheet.png"
    ],
    "matches": ["http://*/*", "https://*/*"]
  }]
}
```

- `storage`：chrome.storage.local 读写设置和 normalized manifest
- `unlimitedStorage`：为 IndexedDB 大资源（多宠物 spritesheet）解除默认配额
- `activeTab`：popup 与当前 tab 交互
- 第一版不加 `tabs` 权限，除非后续需要跨 tab 广播状态
- `web_accessible_resources` 最小暴露，只列内置宠物资源；导入宠物走 IndexedDB 不在此列

### 4.4 宠物资源格式

#### pet.json 标准格式（v2）

```json
{
  "id": "zhengke-youyu",
  "displayName": "zhengke",
  "description": "...",
  "spritesheetPath": "spritesheet.webp",
  "spritesheet": {
    "width": 2304,
    "height": 2592,
    "columns": 9,
    "rows": 9,
    "cellWidth": 256,
    "cellHeight": 288
  },
  "actions": {
    "idle":       { "row": 0, "frameCount": 9 },
    "move-left":  { "row": 1, "frameCount": 9 },
    "move-right": { "row": 2, "frameCount": 9 },
    "greet":      { "row": 3, "frameCount": 9 },
    "working":    { "row": 4, "frameCount": 9 },
    "waiting":    { "row": 5, "frameCount": 9 },
    "success":    { "row": 6, "frameCount": 9 },
    "failed":     { "row": 7, "frameCount": 9 },
    "special":    { "row": 8, "frameCount": 9 }
  },
  "actionMap": {
    "idle": "idle",
    "move-left": "move-left",
    "move-right": "move-right",
    "greet": "greet",
    "working": "working",
    "waiting": "waiting",
    "success": "success",
    "failed": "failed",
    "special": "special"
  }
}
```

字段说明：
- `spritesheet`：所有字段可选，但推荐全部填写。缺失字段通过 preset 或自动检测补全。
- `actions`：每个 action 可选的额外字段：`loop` (boolean)、`next` (string)、`durations` (number[])。
- `actionMap`：可选。缺失时自动通过精确名匹配和 alias 表生成。
- `preset`：可选字符串 `"codex-9x9"` 或 `"hatch-8x9"`，用于补全 spritesheet 缺失字段。

#### 内置 Preset

| Preset | columns | rows | cellWidth | cellHeight |
|--------|---------|------|-----------|------------|
| `codex-9x9` | 9 | 9 | 256 | 288 |
| `hatch-8x9` | 8 | 9 | 192 | 208 |

#### Spritesheet 解析优先级

1. pet.json 有完整 spritesheet 字段（width/height/columns/rows/cellWidth/cellHeight）→ 直接使用
2. pet.json 声明 `preset` → 从 preset 补全
3. 无 preset，但图片 naturalWidth/naturalHeight 能匹配已知 preset → 自动检测补全
4. 无法自动检测 → import UI 让用户手动选择 preset
5. 无法确定 → manifest invalid，拒绝导入

### 4.5 9 个标准 Role 与行为映射

| Role | 触发场景 | 播放方式 | 默认 loop |
|------|---------|---------|-----------|
| `idle` | 默认状态；无拖拽、无 hover、无页面活动 | 循环 | true |
| `move-left` | 拖拽宠物，本次 pointermove dx < 0 | 循环（拖拽期间） | true |
| `move-right` | 拖拽宠物，本次 pointermove dx > 0 | 循环（拖拽期间） | true |
| `greet` | 鼠标 pointerenter 进入宠物区域 | 一次性，结束回 idle（有 5s 冷却） | false |
| `working` | 用户页面活跃（keydown/input/click/scroll/wheel/selectionchange），首次触发后持续刷新 active timer | 循环；3-5s 无新活动后退出 | true |
| `waiting` | 用户 30-60s 无操作 / window blur / document hidden | 循环；恢复活跃后退出 | true |
| `success` | 用户单击宠物（正反馈） | 一次性，结束回 idle（有短冷却） | false |
| `failed` | 窗口 resize 致越界被修正 / 拖拽撞边界 | 一次性，结束回 idle（有短冷却） | false |
| `special` | 双击/右键/长按/长悬停宠物（彩蛋互动） | 一次性，结束回 idle（有冷却） | false |

**默认播放规则**（代码内置，pet.json 可覆写）：
- `idle`、`move-left`、`move-right`、`working`、`waiting` → `loop: true`
- `greet`、`success`、`failed`、`special` → `loop: false, next: "idle"`
- 未知 action（extraActions）→ 默认 `loop: false, next: "idle"`，除非 pet.json 显式覆写

### 4.6 Role → Action 映射解析优先级

1. pet.json 有 `actionMap` → 以 actionMap 为准
2. action 名正好是标准 role 名 → 自动同名映射
3. 使用 alias table 自动匹配：
   - `waving / wave / hello → greet`
   - `running-left / walk-left → move-left`
   - `running-right / walk-right → move-right`
   - `jumping / happy → success`
   - `sad / fail / angry → failed`
   - `dance / special → special`
4. 仍无法确定 → import UI 手动映射
5. 未被映射的 action → `extraActions`，不进入状态机
6. 必要 role（idle）必须有映射，否则拒绝导入

### 4.7 状态机设计

**原则**：纯事件驱动，不持有任何 timer（setTimeout/setInterval）。

**状态优先级**（从高到低）：
1. Dragging (`move-left` / `move-right`)
2. Boundary Failed (`failed`)
3. One-shot (`greet` / `success` / `special` / `failed`)
4. Working
5. Waiting
6. Idle

**One-shot 打断规则**：
- One-shot 播放期间，普通 hook 不打断（userActivity / activitySettled / userInactive / pointerEnter / clickPet / specialTrigger）
- 允许强交互打断：dragStart / dragMove / boundaryFailed / viewportResize → 立即切换
- One-shot 播放结束后，Renderer 发 `animationEnd` hook → 状态机根据 action 的 `next` 规则回到目标 role（默认 idle）

**Hook 列表**（语义层，由下层 Controller 发出）：

| Hook | 来源 | 含义 |
|------|------|------|
| `userActivity` | ActivityTracker | 用户开始活跃（首次 keydown/click 等） |
| `activitySettled` | ActivityTracker | 用户 3-5s 无操作 |
| `userInactive` | ActivityTracker | 用户 30-60s 无操作 / 失焦 / 隐藏 |
| `pointerEnter` | PointerController | 鼠标进入宠物区域 |
| `pointerLeave` | PointerController | 鼠标离开宠物区域 |
| `clickPet` | PointerController | 单击宠物 |
| `specialTrigger` | InteractonGate | 双击/右键/长按/长悬停（已聚合） |
| `dragStart` | DragController | 开始拖拽 |
| `dragMoveLeft` | DragController | 拖拽向左移动 |
| `dragMoveRight` | DragController | 拖拽向右移动 |
| `dragEnd` | DragController | 拖拽结束 |
| `windowBlur` | ActivityTracker | 窗口失焦 |
| `windowFocus` | ActivityTracker | 窗口恢复焦点 |
| `viewportResize` | ViewportObserver | resize 致宠物越界被修正 |
| `animationEnd` | SpritePlayer | One-shot 播放结束 |
| `debugForceState` | Player page (debug) | 强制指定 role |

### 4.8 运行时分层架构

```
Raw Browser Events
  │
  ├─ ActivityTracker ──────────────┐
  │  聚合节流原始事件 (keydown/      │
  │  input/click/scroll/wheel/      │
  │  mousemove/selectionchange/     │
  │  visibilitychange/focus/blur)   │
  │  持有 settle timer (3-5s)       │
  │  持有 inactivity timer (30-60s) │
  │                                 │
  ├─ DragController ───────────────┤
  │  Pointer Events on pet shell   │
  │                                 │
  └─ PointerController ────────────┤
      监听 dblclick/contextmenu/    │
      longPress/hoverDwell          │
                                    ▼
                          InteractionGate
                          冷却过滤 (greet 5s / success /
                          special / failed)
                          多来源聚合 → specialTrigger
                                    │
                                    ▼
                          PetStateMachine
                          纯同步 resolve
                          优先级 + one-shot 锁
                                    │
                                    ▼
                          SpritePlayer
                          逐帧播放 / loop / one-shot
                          播完回调 animationEnd
```

**InteractionGate**：
- 薄 class，只维护 `cooldowns: Map<string, number>`
- `allow(action, now)` → boolean
- `mark(action, now)` → void
- 冷却时长的具体值：greet 5s，success/special/failed 约 2-3s，后续可调
- `toSpecialTrigger(source)` → 聚合 dblclick/contextmenu/longPress/hoverDwell 为 single hook

**ActivityTracker**：
- 不持有 cooldown
- 不负责 special 聚合
- 只负责两件事：
  1. 监听原始浏览器事件，节流后发语义 hook
  2. 管理两个 timer：settle timer (3-5s → `activitySettled`) 和 inactivity timer (30-60s → `userInactive`)

### 4.9 SpritePlayer API

```typescript
class SpritePlayer {
  constructor(options: {
    shell: HTMLElement;
    sprite: HTMLElement;
    spritesheetUrl: string;
    atlas: AtlasSpec;              // columns, rows, cellWidth, cellHeight
    actions: Record<string, ActionSpec>;  // action id → { row, frameCount, loop, next, durations? }
    actionMap: Record<Role, string>;      // role → actual action id
    initialScale?: number;         // default 1
    playbackRate?: number;         // default 1
    frameDuration?: number;        // default 150ms (统一基础时长)
    onComplete?: (info: PlaybackCompleteInfo) => void;
    onFrame?: (info: FrameInfo) => void;
  })

  playRole(role: Role): void;       // 主接口：状态机使用
  playAction(actionId: string): void; // 调试接口：player 页面使用
  pause(): void;
  resume(): void;
  stop(): void;
  setScale(scale: number): void;
  setPlaybackRate(rate: number): void;
  getSnapshot(): FrameInfo;
  destroy(): void;
}
```

- 帧时长：优先用 action 自身的 `durations[frameIndex]`，没有则用统一 `frameDuration`（默认 150ms）
- `playbackRate` 缩放最终时长：`effectiveDuration = baseDuration / playbackRate`
- `playbackRate` 范围 0.25x–2x
- `onComplete` 返回 `{ role?, actionId, next? }`

### 4.10 帧时长策略

- 统一基础帧时长：**150ms**
- 如果 action 有 `durations` 数组，action 的 durations 优先
- 用户通过 popup 或 player 的 speed slider (0.25x–2x) 调节 `playbackRate`
- `effectiveDuration = baseDuration / playbackRate`

### 4.11 存储模型

**chrome.storage.local**（小数据、频繁读写）：

```json
{
  "pets": {
    "zhengke-youyu": {
      "id": "zhengke-youyu",
      "displayName": "zhengke",
      "schemaVersion": 2,
      "spritesheet": { "width": 2304, "height": 2592, "columns": 9, "rows": 9, "cellWidth": 256, "cellHeight": 288 },
      "actions": { ... },
      "actionMap": { ... },
      "extraActions": [],
      "assetKey": "pet:zhengke-youyu:spritesheet"
    }
  },
  "selectedPetId": "zhengke-youyu",
  "settings": {
    "enabled": true,
    "scale": 1,
    "animationSpeed": 1,
    "debugMode": false,
    "forcedState": null
  }
}
```

**IndexedDB**（大资源）：

| Key | Value |
|-----|-------|
| `pet:{id}:spritesheet` | Blob (webp/png) |

**加载流程**：
1. 从 chrome.storage.local 读取 selectedPetId 和 manifest
2. 通过 manifest.assetKey 去 IndexedDB 读取 spritesheet Blob
3. `URL.createObjectURL(blob)` 生成 objectUrl
4. SpritePlayer 使用 objectUrl 播放
5. 切换/卸载宠物时 `URL.revokeObjectURL(objectUrl)`

**extensionApi adapter**：抽象 `chrome.*` 调用，使 UI 页面可在 `vite dev` 下用 mock 数据运行 HMR。

### 4.12 内容脚本注入

- 注入时机：`document_idle`
- 作用域：仅 top window（`window.top === window.self`）
- 排除页面：`chrome://`、`edge://`、`about:`、`chrome-extension://`、Chrome Web Store（`chrome.google.com/webstore`、`chromewebstore.google.com`）
- 覆盖层：Shadow DOM + `petOverlay.css`（fixed positioning、z-index、pointer-events 等）
- 宠物始终在最上层，不与宿主页面 DOM/CSS 冲突

### 4.13 拖拽与边界

- 边界计算使用当前宠物的 `cellWidth`/`cellHeight` × `scale`（动态读取，非硬编码常量）
- 默认位置：右下角，margin 24px
- 拖拽中使用 `setPointerCapture` + `userSelect: none`
- 拖拽结束后自动保存位置到 storage
- Resize 时重新 clamp 位置，若发生越界修正 → 触发 `failed` one-shot

### 4.14 Catalog 生成

- Build 时扫描 `public/pets/` 下所有文件夹
- 读取每个 `pet.json` 的 `id`/`displayName`/`description`
- 生成 `dist/pets/catalog.json`（仅索引信息，不做完整 manifest）
- 完整 manifest 由运行时 `loadPet` 从 pet.json 读取并 normalize

### 4.15 导入流程

1. 用户上传 raw pet.json + spritesheet 图片
2. 系统读取图片 naturalWidth/naturalHeight
3. 补全 spritesheet 参数（完整字段 → preset → 自动检测 → 手动选）
4. 解析 actions 列表
5. 自动生成 actionMap（精确匹配 → alias 表 → 待手动映射）
6. 导入 UI 展示检测结果和 actionMap
7. 用户修正 9 个 role 的映射（下拉选择）
8. 未映射 action → extraActions
9. 用户确认 → 生成 normalized manifest → 存 chrome.storage.local
10. Spritesheet Blob → 存 IndexedDB
11. 第一版不做完整编辑器（不开放 loop/next/durations 编辑），这些走系统默认规则

### 4.16 Content Script → Background 通信

沿用参考项目的消息模式：

- `PET_GET_GLOBAL_STATE`：content script 请求当前全局状态
- `PET_UPDATE_GLOBAL_STATE`：content script/popup 更新全局状态 patch
- `PET_GLOBAL_STATE_UPDATED`：background 广播状态变更到所有 content scripts
- `PET_GET_STATUS`：popup/player 查询当前 pet 状态
- `PET_STATUS_RESPONSE`：返回当前状态快照
- `PET_RESET_POSITION`：重置位置到默认

### 4.17 测试策略

第一版使用 **vitest**，只测纯逻辑函数：

- `normalizePetManifest`：spritesheet 补全、preset 匹配、字段 fallback
- `actionResolver`：role→action 映射、alias table、优先级链
- `PetStateMachine.pickState()`：优先级解析、one-shot 锁、打断规则
- `InteractonGate.allow()`：cooldown 判断、过期逻辑
- `bounds.clampPosition()`：边界计算

不测试：content script 注入、background SW、chrome.storage、IndexedDB、拖拽交互（这些需要 Chrome Extension 运行环境，第一版用实际加载扩展手动验证）。

## 5. Non-Goals (v1 不做)

- 不做跨 tab 状态同步（需要 `tabs` 权限的"广播到所有 tab"逻辑第一版保留）
- 不做宠物音频/音效
- 不做宠物对话气泡/文字展示
- 不做宠物"情绪值"或"好感度"系统
- 不做宠物自动行走/寻路（宠物只跟随拖拽移动，不会自动在页面上行走）
- 不做云同步（设置和宠物只在本地存储）
- 不做完整的 pet manifest 编辑器（import 页面只做映射确认，不开放 loop/next/durations 编辑）
- 不做 iframe 内嵌页面的宠物注入
- 不做移动端支持
- 不引入 `tabs` 权限（除非后续明确需要）

## 6. Acceptance Criteria

### AC-1: 宠物在网页上正常播放
- 打开任意 HTTP/HTTPS 页面，宠物出现在右下角
- 宠物根据用户交互自动切换动画（idle/working/waiting/greet/success/failed/special/move-left/move-right）
- 宠物可拖拽，拖拽时不被选中文字，放下后位置正确保存
- 宠物不干扰页面正常交互（点击穿透、表单输入等）
- 切换标签页/隐藏窗口后宠物进入 waiting，回来后退出 waiting

### AC-2: Popup 控制面板可用
- 点击扩展图标打开 popup
- Toggle 能正确启用/禁用宠物
- Scale 滑块实时生效（0.5x–2x）
- Speed 滑块实时生效（0.25x–2x）
- 切换宠物实时生效
- 重置位置按钮有效
- 导航到 player/import 页面正确

### AC-3: Player 播放器可调试
- 可选择任意已安装宠物
- 可逐一触发 9 个 role，动画正确播放
- 可手动触发状态机 hook
- Scale/speed 实时生效
- 显示 spritesheet 参数、actionMap、当前帧信息
- 背景切换正常

### AC-4: Import 导入流程完整
- 上传 pet.json + spritesheet 图片成功
- Spritesheet 参数自动检测/补全正确
- ActionMap 自动生成并展示
- 用户可手动修正映射
- 保存后宠物出现在 popup 列表中
- 内置 preset (codex-9x9 / hatch-8x9) 正确工作

### AC-5: 旧格式兼容
- hatch-8x9 格式的旧宠物可通过 preset 正常导入和播放
- codex-9x9 格式的新宠物可直接使用

### AC-6: 存储正确
- chrome.storage.local 存元数据，IndexedDB 存 spritesheet Blob
- 多宠物切换时正确加载和卸载
- Object URL 正确创建和回收，无内存泄漏

## 7. Verification Plan

### 7.1 自动化测试 (vitest)

| 测试对象 | 用例 |
|---------|------|
| `normalizePetManifest` | 完整字段 pass-through / preset 补全 / 自动检测 / 缺失字段报错 / 未知 preset 报错 |
| `actionResolver` | 精确名匹配 / alias 匹配 / actionMap 优先 / 未知 action fallback / 多对一映射 |
| `PetStateMachine.pickState()` | 9 个优先级场景 / one-shot 锁 / drag 打断 one-shot / animationEnd 回到 idle |
| `InteractonGate.allow()` | 冷却期内拒绝 / 冷却期后允许 / mark 更新 timestamp |
| `bounds.clampPosition` | 各种 viewport/scale 组合 / 越界修正 / 边界等于 viewport |

### 7.2 手动验证

| 场景 | 验证方式 |
|------|---------|
| 宠物注入任意网页 | 加载扩展，访问 google.com / github.com / wikipedia.org |
| 拖拽 | 拖动宠物到各角落，确认跟随、边界限制、位置保存 |
| 状态自动切换 | 操作页面（打字/滚轮/点击），观察宠物动画切换 |
| Waiting 进入/退出 | 切到其他 tab 等待 30s+，切回观察 |
| Popup 所有控件 | 打开 popup，操作每个 slider/按钮，确认实时生效 |
| Player 调试 | 打开 player.html，逐一触发 role，检查动画和 spritesheet 参数 |
| Import 完整流程 | 上传测试用 pet.json + spritesheet，完成映射，确认新宠物可用 |
| Preset 兼容 | 导入一个 hatch-8x9 旧格式宠物，确认正常播放 |
| 内置宠物切换 | 在 popup 中切换 zhengke-youyu → 其他内置宠物 → 再切回 |
| 特殊交互 | 双击/右键/长悬停宠物，确认 special 动画播放 |
