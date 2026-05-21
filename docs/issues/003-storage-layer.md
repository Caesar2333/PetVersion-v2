# #3 — 存储层

**Type:** AFK
**Blocked by:** #2（类型定义）
**估时:** 3-4h

---

## What to build

抽象 `chrome.storage.local` + `chrome.tabs` + `chrome.runtime` 为 `extensionApi` 接口。实现真实适配器（Chrome 环境）和 mock 适配器（vite dev 环境）。实现 chrome.storage.local 的 settings/pet 列表 CRUD + IndexedDB 的 Blob 存取。管理 Object URL 生命周期。

## 涉及模块

- `src/storage/extensionApi.ts`
  - `ExtensionApi` 接口：`getStorage`, `setStorage`, `createTab`, `sendMessage`, `onMessage`
  - `createChromeApi()` — 真实 chrome.* 包装
  - `createMockApi()` — 内存 Map 实现，供 vite dev 使用
  - 通过环境变量或 import.meta 自动选择实现

- `src/storage/chromeStorage.ts`
  - `getSettings(): Promise<Settings>`
  - `updateSettings(patch): Promise<void>`
  - `getPetList(): Promise<PetListItem[]>`
  - `getPetManifest(id): Promise<NormalizedManifest>`
  - `addPetManifest(manifest): Promise<void>`
  - `removePet(id): Promise<void>`
  - `getSelectedPetId(): Promise<string>`
  - `setSelectedPetId(id): Promise<void>`
  - 存储结构见 PRD §4.11

- `src/storage/indexedDbAssets.ts`
  - `storeAsset(key, blob): Promise<void>`
  - `getAsset(key): Promise<Blob | null>`
  - `removeAsset(key): Promise<void>`
  - `clearAll(): Promise<void>`

- Object URL 生命周期：
  - `createObjectUrl(blob): string`
  - `revokeObjectUrl(url): void`
  - 切换/卸载宠物时调用 revoke

## Acceptance criteria

- [ ] mock 模式下：写 settings → 读回一致 → 更新 patch → 读回已更新 → 删除 → 确认空
- [ ] mock 模式下：存 Blob → 读回一致 → 删除 → 读回 null
- [ ] mock 模式下：add pet manifest → pet list 包含新条目
- [ ] Object URL create + revoke 配对正确（手动追踪）
- [ ] 类型安全：所有 CRUD 操作返回强类型 Promise

## 禁止改动范围

- 不实现 UI
- 不实现消息广播逻辑（那是 #8 的范围）

## 验证方式

```bash
# 单元测试（mock 模式）
npx vitest run src/storage/

# 手动：在 vite dev 下 console 测试 CRUD
```
