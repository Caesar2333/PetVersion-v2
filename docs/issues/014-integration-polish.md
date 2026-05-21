# #14 — 端到端集成 & 打磨

**Type:** HITL（视觉审阅、微交互手感调校）
**Blocked by:** 所有前置切片（#1–#13）
**估时:** 4-6h

---

## What to build

将所有模块端到端串通，确保 content script → 状态机 → SpritePlayer → popup 控制整条链路正确工作。实现设计规格中的微交互层（"灵动"细节）。暗色模式全线支持。响应式布局验证。键盘导航收尾。

## 涉及工作

### 1. 端到端串联

- Content script 启动 → 从 storage 加载 pet → 实例化 SpritePlayer → idle 播放
- ActivityTracker + InteractionGate → hooks → StateMachine → SpritePlayer.playRole()
- DragController → hooks + 位置更新 + 持久化
- Popup → 读 storage → 发送 message → content script 响应
- Player → 独立 SpritePlayer 实例 → 调试面板实时更新
- Import → 上传 → 检测 → 存储 → popup 可见

### 2. 微交互（design.md §5.5）

- [ ] Pet thumbnail hover bounce（hover 时 translateY -4px + ease-out-bounce）
- [ ] Toggle 动画：pet icon 跟随 toggle 状态缩放脉冲
- [ ] Slider snap：达到 1× 时 thumb 脉冲一次
- [ ] Step completion：导入步骤连接线从左到右彩色扫过（400ms）
- [ ] Role trigger glow：点击 role 按钮 → viewport border 短暂 accent 发光（200ms）
- [ ] Import success dot burst：确认按钮 burst 8-12 个彩色小点（纯 CSS keyframe，不引入 confetti 库）
- [ ] Pet name text-shadow glow：active 状态下 popup header pet 名字有 accent 色 text-shadow

### 3. 暗色模式全线

- `.dark` class on `<html>` → 所有组件切换暗色 token
- 系统 `prefers-color-scheme` 自动检测
- 手动 toggle 覆写，存入 localStorage
- Player 页面背景切换（checkerboard 等）独立于主题

### 4. 响应式布局验证（design.md §6）

- Popup：≥380px 3 列 pet grid，<380px 2 列
- Player：≥1024px 双栏，≥640px 堆叠，<640px 折叠面板
- Import：≥640px 居中卡片，<640px 全宽

### 5. 键盘导航（design.md §5.3）

- Popup：Tab 顺序 toggle → sliders → buttons
- Player：Tab 所有控件，Arrow 键调 slider（±0.05），数字键 1-9 触发对应 role
- Import：Tab 表单，Enter 前进，Escape 后退

### 6. 性能检查

- 宠物注入不阻塞宿主页面加载
- SpritePlayer 60fps，无 layout thrashing
- Object URL 正确回收，无内存泄漏
- Spritesheet Blob 不重复加载

## Acceptance criteria

完整按 PRD §7.2 手动验证清单执行：

- [ ] 场景 1：宠物注入任意网页 → 动画正常，不干扰宿主页面
- [ ] 场景 2：拖拽到各角落 → 跟随 + 边界限制 + 位置保存
- [ ] 场景 3：操作页面（打字/滚轮/点击）→ 宠物动画自动切换
- [ ] 场景 4：切 tab 等待 30s+ → 切回 → waiting 进入/退出
- [ ] 场景 5：Popup 所有控件操作 → 实时生效
- [ ] 场景 6：Player 逐一触发 role → 动画和数据正确
- [ ] 场景 7：Import 完整流程 → 新宠物可用
- [ ] 场景 8：hatch-8x9 旧格式通过 preset 导入 → 正常播放
- [ ] 场景 9：popup 切换内置宠物 → 正常
- [ ] 场景 10：双击/右键/长悬停 → special 动画播放

视觉检查：

- [ ] 与 `design/design-preview.html` 参考无视觉回归
- [ ] 微交互自然流畅，不突兀
- [ ] 暗色模式下所有组件颜色正确
- [ ] 无控制台错误
- [ ] 键盘操作流畅

## 禁止改动范围

- 不新增功能（这是打磨切片，不是功能切片）
- 不修改核心逻辑架构

## 验证方式

加载扩展 → 按 PRD §7.2 清单逐项验证。对比 `design/design-preview.html` 做视觉审查。
