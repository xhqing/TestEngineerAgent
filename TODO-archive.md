# TODO 归档（Hopper · 软件测试工程师）

> 已处理条目的归档（✅**已完成** / ✅**已更新** / ✅**已放弃**），保留原正文与编号，永不复用。看到 ✅ 条目即跳过。

## 🟠 橙色紧急度

- [ ] **T6** 为 ghostty-launcher（Atlas 项目）Issue #1 出测试（先红），写入 worktree `~/Developer/ghostty-launcher-new-window-reliability`（记录：2026-09-29 19:33）✅**已完成**（完成：2026-09-29 21:45）
  **背景**：ghostty-launcher 面板的「New Window」按钮偶发无反应（2026-09-29 发生一次、无法按需复现）。排查结论：不是调用失败——系统日志里发生时段扩展没有任何调用记录，是**点击没有变成扩展调用**；且扩展自身零日志（所有失败路径静默吞掉），根因无法定位。另有多处代码审查发现的竞态隐患：列表每 2s 全量重建 DOM（点窗口行可能被吞、估算 3–7%/次）、面板重开时有约 0.5–1s 空白期、运行中 New Window 调用失败会静默 fallback 到 `open -na`（会拉起第二实例）、轮询无在途守卫。修复触及 `extension.js` 运行行为，属核心开发，按 dev-workflow 测试先行。**该仓库是本团队首个走 dev-workflow 的项目：目前无测试框架、无测试命令、无 CI（无 `.github/`）、main 无分支保护**——本任务一并把测试框架与测试命令建立起来（CI 与分支保护随后补齐；顺序必须是先 ci.yml 经 PR 进 main → 再开保护设 required，反序死锁）。
  **要做什么**：① 读 Issue #1（https://github.com/xhqing/ghostty-launcher/issues/1）正文与评论——评论里有 dev 侧给的接口提示：拟抽 `lib/ghostty.js`（纯逻辑：JXA 脚本生成 / 决策与重试 / `shellQuote`，外部依赖可注入）与 `media/panel.js`（webview 端脚本，纯函数段带 `module.exports` 守卫、node 可直接 require）；② 选测试框架（零依赖方案：Node 内置 `node:test`），在项目正式测试位置（`test/`）写测试、带 `# TEST_CASES_WRITE_OK` 标记写入；③ 自跑确认红（实现尚不存在）；④ 定下测试命令（`package.json` 的 `test` script）——测试命令配置属测试体系、由本侧定义，本地全量与 CI 都跑它；⑤ 按 dev-workflow `references/test-cases.md`「CI 集成」规范拟 `ci.yml`（触发器含 `pull_request`(main) + `push`(main)，执行内容覆盖全量测试 + 类型 / 语法检查）；⑥ 交付自检全过后报告应 `git add` 的路径与改动摘要（commit 由用户亲自执行，AI 不代行）。
  **关联**：Issue #1（ghostty-launcher）；开发侧 Atlas 已建 worktree + 分支 `fix/new-window-reliability`（从 main `0b0af91` 开出，2026-09-29 18:38；Issue #1 已建、含排查结论与 5 条期望行为）。测试就位后 Atlas 实现到绿 → 本地全量绿 → PR（`fixes #1`）→ CI 绿 auto-merge → 预发布试用验收。本任务为 2026-09-29 跨 agent 下发新约定（写入对方 TODO + 报编号）的首个用例。
  **完成说明**（2026-09-29 21:45）：①～⑥ 全部完成——交付 `test/ghostty.test.js`（18 用例）+ `test/panel.test.js`（7 用例，零依赖 Node `node:test`）、`package.json` 的 `test` / `check` scripts、`.github/workflows/ci.yml`（`pull_request`(main) + `push`(main)，语法检查 + 全量测试）、`.vscodeignore` 排除 `test/**` `.github/**`、ghostty-launcher `CHANGELOG.md` 记录；验证：`npm run check` 全过、`npm test` 全红（`lib/ghostty.js` / `media/panel.js` 实现未落地、先红有效），另做 stub 自检全绿（25/25）+ 三类变异红（不重试 3 / 回退 open 2 / 签名忽略 running 1，排除永真断言），自检临时物已清理。测试接口契约按 Issue 评论提示：`lib/ghostty.js` 导出 `LIST_INTERVAL`（3000）/ `shellQuote` / `buildListScript` / `buildActivateScript` / `buildNewWindowScript` / `runNewWindow` / `runActivate`；`media/panel.js` 导出 `signatureOf` / `hintFor`。已向用户报告应 `git add` 的路径（commit 由用户亲自执行）。后续：Atlas 实现到绿 → 本地全量绿 → PR（`fixes #1`）→ CI 绿 auto-merge → 合并后开 main 分支保护设 required（先 CI 进 main、再开保护）。

## 🟢 绿色紧急度

- [ ] **T2** 与各开发 agent 项目建立「用例先行」任务下发机制（记录：2026-09-06 12:36）✅**已更新**（更新：2026-09-21 21:02）
  **背景**：三权分立设计里新需求要先经 Hopper 转验收用例再交开发 agent 实现，但这条协作链路目前只在方法论层面成立，还没有落到各开发 agent 项目的工作流里（它们的 CLAUDE.md 里还没有「测试纪律」相关约束）。
  **要做什么**：① 在 Anvil / Atlas / Hermes / Alfred 等开发 agent 项目的 CLAUDE.md 里加测试纪律条目（不许改用例迁就实现、有异议上报裁决、交付前本地全量跑测试）；② 与 Kit 约定新需求任务单的模板——需求描述 + Hopper 产出的验收用例 +「PR 全绿才可合并」的完成标准；③ 第一个真实新需求走通全链路后固化流程。
  **更新说明**：dev-workflow 2026-09-21 三次修订后，本条目标已被更强的机制实现，原计划动作不再需要——①「各项目 CLAUDE.md 加测试纪律」由全局 dev-workflow skill（所有 agent 会话共享，第 3 步测试先行 + 第 4 步开发纪律）+ 跨端测试文件写保护 hook（工具层硬强制，比文本纪律更强）取代；②「任务单模板」由 GitHub Issue（复现步骤 / 预期行为）+ `fixes #N` 状态机取代；③「第一个真实需求走通全链路」待实际需求发生时自然验证，不再单列待办。存量 test-cases/ 与本项目 cases/ 的处置另立 T3 跟踪。
