# #1 — 项目脚手架 & 构建系统

**Type:** AFK
**Blocked by:** None
**估时:** 3-4h

---

## What to build

Vite 7 + React 19 + TypeScript + Tailwind CSS v3 + shadcn/ui 项目初始化。5 入口 `rollupOptions.input`（`content.ts`, `service-worker.ts`, `popup.html`, `player.html`, `import.html`）。`globals.css` 写入 OKLch 设计 token。Chrome Extension MV3 `manifest.json`。`vite dev` 下 mock `extensionApi` 让 UI 页面不依赖 Chrome HMR 开发。`vite build` 产出完整可加载的解包扩展。

## 涉及模块

- `vite.config.ts` — 多入口 rollupOptions、alias、dev mock 注入
- `tsconfig.json` — paths、strict
- `tailwind.config.ts` — shadcn/ui preset、自定义 OKLch token
- `src/styles/globals.css` — `@tailwind` 指令 + `:root` / `.dark` CSS 变量
- `components.json` — shadcn/ui 配置
- `public/manifest.json` — MV3 permissions、web_accessible_resources
- `src/shared/constants.ts` — 全局常量
- `index.html` (popup), `player.html`, `import.html` — 各页面 HTML shell
- `package.json` — dependencies、scripts（`dev` / `build` / `typecheck`）

## Acceptance criteria

- [ ] `pnpm dev` 启动，popup/player/import 三个 HTML 页面可分别访问，展示空白 Tailwind 页面
- [ ] `pnpm build` 成功退出，`dist/` 包含 5 个入口产物 + manifest.json
- [ ] `globals.css` 包含完整 :root + .dark OKLch token
- [ ] shadcn/ui Button 组件可在任一页面正常渲染
- [ ] TypeScript strict mode 通过，无类型错误
- [ ] `dist/manifest.json` 包含正确的 permissions 和 web_accessible_resources

## 禁止改动范围

- 不写任何宠物业务逻辑
- 不引入 CRXJS

## 验证方式

```bash
pnpm dev          # 浏览器分别访问 /popup.html /player.html /import.html
pnpm build        # 确认 dist/ 结构正确
pnpm typecheck    # 确认零错误
```
