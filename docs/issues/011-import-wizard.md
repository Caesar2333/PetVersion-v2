# #11 — Import 页面：完整导入向导

**Type:** AFK
**Blocked by:** #1（构建）、#2（类型）、#3（存储）
**估时:** 6-8h

---

## What to build

独立的 import.html 全页导入向导。3 步流程：上传文件 → 审核检测结果和 action map → 确认并保存。设计参考 `design/design-preview.html` 的 import 部分和 `design/design.md` §3.3。

## 涉及模块

`src/app/import/`

### 整体布局

```
┌──────────────────────────────────────┐
│  Step indicator                      │
│  ● Upload ── ○ Review ── ○ Confirm   │
├──────────────────────────────────────┤
│  Step content area                   │
├──────────────────────────────────────┤
│  [← Back]              [Next →]      │
└──────────────────────────────────────┘
```

### 子组件

- `ImportApp.tsx` — 根组件，管理 3 步状态、文件数据、检测结果、manifest 生成
- `StepIndicator.tsx`
  - 3 个连接圆点 + 标签
  - Completed：绿色 + ✓
  - Current：accent 填充 + pulsing ring
  - Future：灰色 border + 数字
  - 连接线：完成步之间绿色填充
- `Step1Upload.tsx`
  - Drop zone：2px dashed border, hover accent + bg tint
  - 隐藏 `<input type="file" accept=".json,.webp,.png">`
  - 文件 chips：文件名 + 大小 + 删除按钮
  - 校验：必须有 pet.json + 一张图片
  - 拖拽支持：dragover/dragleave/drop
  - 解析 pet.json JSON → 验证必填字段（id）
- `Step2Review.tsx`
  - Detection card：Preset（auto/manual）、Image size、Grid、Cell size、Actions count
  - ActionMap 表格：Role | Action（mono）| Status badge
  - Unmapped role 行 → `<select>` 下拉选择可用 action
  - Extra actions 列表（底部灰色小字）
  - 状态徽章：✓ auto（green）/ ↻ alias（amber）/ ✗ unmapped（red）/ ✎ manual（blue）
- `Step3Confirm.tsx`
  - Import summary card：Pet ID、Display name、Preset、Actions count、Storage target
  - "Import Pet" primary button
  - 点击 → normalize manifest → `chrome.storage.local` + IndexedDB → success toast → 重置向导
- `ImportNav.tsx` — Back / Next 按钮，Next 在 step 3 变为 "Confirm"
- `Toast.tsx` — Sonner toast：success/error，3s 自动消失

### 检测逻辑（在 Step 1 → Step 2 过渡时执行）

1. 读取图片 naturalWidth/naturalHeight
2. Spritesheet 参数补全（调用 #2 的 normalizePetManifest）：
   - pet.json 有完整字段 → 直接使用
   - 有 preset → 补全
   - 图片尺寸匹配已知 preset → 自动检测
   - 否则 → 展示错误，让用户手动选择 preset
3. Action 解析 → actionResolver（#2）生成 actionMap
4. 展示结果

### 存储逻辑（Step 3 确认时）

1. Normalize manifest → 生成 `NormalizedManifest`
2. Spritesheet 图片 → Blob → `indexedDbAssets.storeAsset(assetKey, blob)`
3. Manifest（不含 blob）→ `chromeStorage.addPetManifest(manifest)`
4. 如果这是第一只宠物 → 自动设为 selected
5. Success toast + 3s 后重置向导

## Acceptance criteria

- [ ] Step 1：拖拽或点击上传 pet.json + spritesheet → 显示文件 chips
- [ ] 上传非法 JSON → 红色错误提示
- [ ] 上传 pet.json 缺 id → 错误提示
- [ ] 仅上传一个文件 → Next 按钮禁用
- [ ] Step 2：检测结果准确显示 preset、图片尺寸、网格、cell 尺寸、actions 数量
- [ ] Action map 表格正确标注每个 mapping 状态
- [ ] Unmapped role 可通过下拉手动选择 action
- [ ] Extra actions 正确列出
- [ ] Step 3：摘要信息正确
- [ ] 点击 Import → 保存到 chrome.storage.local + IndexedDB
- [ ] 保存后 popup 的 pet 列表出现新宠物
- [ ] 切换新宠物 → 页面正常播放
- [ ] 步骤指示器实时反映当前步骤
- [ ] Back/Next 导航正常
- [ ] Toast 通知正常显示和消失

## 禁止改动范围

- 不做完整 manifest 编辑器（不开放 loop/next/durations 编辑，走默认规则）
- 不做 spritesheet 裁剪/编辑功能

## 验证方式

准备测试用 pet.json + spritesheet → 打开 import.html → 完成 3 步流程 → 确认宠物出现在 popup → 切换到新宠物 → 确认页面播放正常。

测试异常路径：非法 JSON、缺 id、未知 spritesheet 尺寸 → 确认错误提示正确。
