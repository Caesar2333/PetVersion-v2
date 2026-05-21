# #12 — 内置宠物资源 & Catalog 生成

**Type:** HITL（需要实际 spritesheet 美术资源）
**Blocked by:** #2（manifest 格式）
**估时:** 2-3h（不含美术制作时间）

---

## What to build

将内置宠物文件夹放入 `public/pets/`，每个文件夹包含 `pet.json`（v2 格式）+ `spritesheet.webp`/`.png`。构建脚本自动扫描并生成 `dist/pets/catalog.json`。运行时 `loadPet` 根据来源加载 manifest + Blob。

## 涉及模块

### 内置宠物目录结构

```
public/pets/
  zhengke-youyu/
    pet.json          ← v2 格式，完整 spritesheet + actions + actionMap
    spritesheet.webp  ← 9×9 × 256×288 = 2304×2592
  [其他内置宠物]/
    pet.json
    spritesheet.webp
```

### pet.json 模板（zhengke-youyu）

```json
{
  "id": "zhengke-youyu",
  "displayName": "zhengke",
  "description": "Codex 9×9 default pet",
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
    "idle": { "row": 0, "frameCount": 9 },
    "move-left": { "row": 1, "frameCount": 9 },
    "move-right": { "row": 2, "frameCount": 9 },
    "greet": { "row": 3, "frameCount": 9 },
    "working": { "row": 4, "frameCount": 9 },
    "waiting": { "row": 5, "frameCount": 9 },
    "success": { "row": 6, "frameCount": 9 },
    "failed": { "row": 7, "frameCount": 9 },
    "special": { "row": 8, "frameCount": 9 }
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

### Catalog 生成脚本

- `scripts/generate-catalog.ts`（或作为 vite plugin）
  - 扫描 `public/pets/*/pet.json`
  - 提取 `id` / `displayName` / `description`
  - 生成 `dist/pets/catalog.json`：
    ```json
    [
      { "id": "zhengke-youyu", "displayName": "zhengke", "description": "..." },
      ...
    ]
    ```
  - 作为 `vite build` 的一部分运行

### 运行时加载

`src/pet-core/repository/loadPet.ts`
- `loadPet(id: string): Promise<{ manifest: NormalizedManifest, objectUrl: string }>`
  - 先查 chrome.storage.local → 找到 manifest
  - 通过 manifest.assetKey → IndexedDB 获取 Blob → `URL.createObjectURL`
  - 内置宠物：pet.json 通过 `chrome.runtime.getURL` 加载；spritesheet 也通过扩展 URL 加载（首次 → 转 Blob 存入 IndexedDB）
  - 返回 normalized manifest + objectUrl

### web_accessible_resources

manifest.json 中暴露：
```json
"web_accessible_resources": [{
  "resources": [
    "pets/catalog.json",
    "pets/*/pet.json",
    "pets/*/spritesheet.webp",
    "pets/*/spritesheet.png"
  ],
  "matches": ["http://*/*", "https://*/*"]
}]
```

## Acceptance criteria

- [ ] `public/pets/` 下至少有一个完整的内置宠物文件夹
- [ ] `vite build` 后 `dist/pets/catalog.json` 存在且内容正确
- [ ] 扩展加载后 popup Pets tab 显示内置宠物
- [ ] 选中内置宠物 → 页面正常播放动画
- [ ] 内置宠物 spritesheet 通过 web_accessible_resources 可访问
- [ ] 切换回内置宠物无需重新导入

## 禁止改动范围

- 不做 pet 编辑器
- 不做宠物商店/在线下载

## 验证方式

`vite build` → 检查 `dist/pets/catalog.json` → Chrome 加载扩展 → popup 确认显示 → 切换宠物 → 页面动画正常。
