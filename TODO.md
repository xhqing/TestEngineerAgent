# TODO（Hopper · 软件测试工程师）

> 活跃待办清单。条目按紧急度分节，节内按记录时间排序。处理完毕移入 `TODO-archive.md`。
> 个人隐私类待办写 `TODO.local.md`（不入 git），编号与公开版共用同一套空间。

## 🔴 红色紧急度

（暂无）

## 🟠 橙色紧急度

- [ ] **T1** 摸底全部产软件项目的测试现状并按痛的程度建回归防护网（记录：2026-09-06 12:36）
  **背景**：本项目因回归痛点立项（改 A 坏 B 反复发生），但各项目的测试现状未实测过——有没有测试框架、有没有 CI、发布前跑不跑测试，一概不清。不知道缺口在哪，防护网就无从下手。
  **要做什么**：① 逐个摸底七个产软件项目（zcode-cli、zcode-vsce、CC-BRIDGE、ResourceMonitor、XPilot、gridtrader、AgentCortex）：测试框架有无、CI workflow 有无（口径按 2026-09-21 dev-workflow：触发器须含 `pull_request` + `push: main`、执行内容须覆盖全量测试 + 类型检查）、发布流程里有没有测试关卡；② 按回归痛的程度排序（改得最频繁、被改坏最多的优先）；③ 最痛的项目先建：搭测试框架（Node 系 vitest / jest，Python 系 pytest）→ 从历史回归事故补测试（先红后绿）→ 补齐 / 恢复启用 ci.yml 并设 required → main 配分支保护（先 CI 进 main、再开保护设 required，反序死锁）；④ 逐项目推进，模式跑顺后推广到其余项目。

## 🟡 黄色紧急度

- [ ] **T3** 处置本项目 `cases/` 存量与各项目 `test-cases/` 存量（记录：2026-09-21 21:02）
  **背景**：dev-workflow 2026-09-21 三次修订取消了「权威源 cases/ 仓库 → 项目 test-cases/ 镜像」单向分发机制与 pending/passed 目录状态机（项目仓库成为唯一记录），本项目 `cases/zcode-cli/pending/` 还压着两个未完成需求组（custom-env-rename、model-picker-env-prefix），各存量项目可能也有 `test-cases/` 旧目录——不处置的话，权威源已废、镜像语义已变，两边都会误导后续工作。
  **要做什么**：① 本项目 `cases/zcode-cli/pending/` 两组：与用户确认旧需求是否仍要做——仍要做则到 zcode-cli 仓库开 Issue 承接（正文可用原 requirement.md 内容）、用例按新体系重出（正式测试位置 + 先红后绿），不做则记录后清理；② 摸底各产软件项目的 `test-cases/` 存量：`passed/` 用例按项目测试框架搬进正式测试位置（带 `# TEST_CASES_WRITE_OK` 标记整组搬移、适配 import 路径），迁移经 PR + CI 验证全绿后删旧目录；`pending/` 未完成组同①处置；③ 全部处置完后删除本项目 `cases/` 目录（机制已取消，留着是误导），CHANGELOG 记录收尾。

## 🟢 绿色紧急度

- [ ] **T4** 为 pi fork 的 Issue #4 出测试（先红），写入 Worktree `~/Developer/pi-fork-update-check`（记录：2026-09-21 21:08）
  **背景**：用户在 pi fork（xhqing/pi）上提了 Issue #4——fork 之后 update check 与 pi update 仍指向 upstream `pi.dev`，要求修复前先出测试。按 dev-workflow 测试先行流程（2026-09-21 修订版），测试 Agent 在功能分支上先出题、自跑见红，开发 agent 才开工实现到绿。（本条曾于 21:01 以 T3 编号记录，后被另一会话的并发 TODO 重写覆盖丢失，无归档痕迹，21:08 按新流程重写恢复并顺延编号为 T4；T3 现由「处置 cases/ 存量」占用。）
  **要做什么**：① 读 Issue #4 正文与评论，弄清预期行为（update check 指向哪、更新命令从哪拉取版本）；含糊处在 Issue 评论里与用户澄清，不猜——Issue 正文就是需求文档与验收标准，不再另写 requirement.md；② 用 pi 项目现有测试框架在项目正式测试位置（项目测试目录惯例位置）写测试，写入带 `# TEST_CASES_WRITE_OK` 授权标记；③ 自跑确认红（实现尚不存在，断言必红——验证断言有效，防永真断言）；④ 经用户明确授权后 commit 进该功能分支。不写 cases/ 权威源、不建 test-cases/ 目录——该体系已于 2026-09-21 随 Issue 状态机一并取消，项目仓库就是唯一记录（Issue + 测试都在 GitHub 上）。
