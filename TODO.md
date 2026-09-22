# TODO（Hopper · 软件测试工程师）

> 活跃待办清单。条目按紧急度分节，节内按记录时间排序。处理完毕移入 `TODO-archive.md`。
> 个人隐私类待办写 `TODO.local.md`（不入 git），编号与公开版共用同一套空间。

## 🔴 红色紧急度

（暂无）

## 🟠 橙色紧急度

- [ ] **T1** 摸底全部产软件项目的测试现状并按痛的程度建回归防护网（记录：2026-09-06 12:36）
  **背景**：本项目因回归痛点立项（改 A 坏 B 反复发生），但各项目的测试现状未实测过——有没有测试框架、有没有 CI、发布前跑不跑测试，一概不清。不知道缺口在哪，防护网就无从下手。
  **要做什么**：① 逐个摸底七个产软件项目（zcode-cli、zcode-vsce、CC-BRIDGE、ResourceMonitor、XPilot、gridtrader、AgentCortex）：测试框架有无、CI workflow 有无（口径按 2026-09-21 dev-workflow：触发器须含 `pull_request` + `push: main`、执行内容须覆盖全量测试 + 类型检查）、发布流程里有没有测试关卡；② 按回归痛的程度排序（改得最频繁、被改坏最多的优先）；③ 最痛的项目先建：搭测试框架（Node 系 vitest / jest，Python 系 pytest）→ 从历史回归事故补测试（先红后绿）→ 补齐 / 恢复启用 ci.yml 并设 required → main 配分支保护（先 CI 进 main、再开保护设 required，反序死锁）；④ 逐项目推进，模式跑顺后推广到其余项目。

- [ ] **T5** 修复 pi fork PR #5 测试提交的遗留缺陷（2 条旧用例残留 + 3 个类型错误），解锁 Issue #4 修复合并（记录：2026-09-22 11:00）
  **背景**：2026-09-21 为 pi fork Issue #4 出的测试提交（xhqing/pi 分支 `fix/fork-update-check`，commit 698323171，PR #5，`fixes #4`）复盘发现四处执行遗漏：① 测试文件注释、pi 仓库 CHANGELOG、本项目 T4 进展三处均声称「删 renamed-package 两用例」，实际只删一条——`fails self-update when renamed npm package installation fails` 仍在文件里，mock 还是旧 `{packageName, version}` 载荷；② 同文件 `keeps npm self-updates non-managed when the managed environment is inherited` 的 mock 是全文件唯一没扫成 `tag_name` 形态的，且断言 `PACKAGE_NAME@VERSION` registry 安装规格与 Issue #4「切断官方包覆盖路径」的预期矛盾；③ 两个测试文件里 3 个 TS2493 类型错误（fetch mock 零参签名 + `mock.calls[0]` 解构）。PR #5 的 CI（run 35609928521）已在 Check 步骤红于 ③，Test 步骤未跑；①② 在开发 agent 已完成实现（Issue #4 三项预期行为均已实现，本地验证 29/31，唯二红即这两条）后必红。开发侧停在此等测试侧修复，PR 合并被整体卡住。
  **要做什么**（worktree `~/Developer/pi-fork-update-check`，测试文件写入带 `# TEST_CASES_WRITE_OK`）：① 删 `packages/coding-agent/test/package-command-paths.test.ts` 里 `fails self-update when renamed npm package installation fails` 用例（其上方注释与 pi CHANGELOG 均已声称删除；fork 检查源是 GitHub Releases，载荷无 packageName 字段，改名迁移流程不可达）；② 同文件 `keeps npm self-updates non-managed when the managed environment is inherited`：mock 改为 `tag_name` 形态（值 `v` + VERSION），断言里 `PACKAGE_NAME@VERSION` 改为 fork Release tarball URL（实现入口 `getForkReleaseTarballUrl`，即 Issue #4 许可的「指向 fork 自己安装源」方案）；③ 修 3 个 TS2493：两处 fetch mock 签名声明 input 参数（`vi.fn(async (_input: string | URL | Request) => ...)`）使 `mock.calls[0]` 解构类型成立；④ 改完自跑：pi 仓库 `npm run check` 过 + `test/version-check.test.ts`（9/9）与 `test/package-command-paths.test.ts`（31/31）全绿（实现已在分支工作区，直接对跑即先绿验证）；⑤ 用户亲自 commit（自行 `git add` + `/commit` skill：commit + push + PR auto-merge），CI 绿即 auto-merge 合并、`fixes #4` 自动关 Issue（2026-09-22 用户定：AI 不申请、不代行 commit）。
  **进展**：①～④ 已完成（2026-09-22 11:36）——六处修复全部落盘并逐项枚举复验零残留：删残留用例（it 块整删）、inherited-env 用例 mock 改 `tag_name: v${VERSION}` + 断言改 `getForkReleaseTarballUrl(VERSION)`（import 实现入口）、两处零参 fetch mock 补签名（version-check 处带 `init?: RequestInit` 以支撑双元素解构）；`npm run check` 全过（3 个 TS2493 清零），两文件 38/38 全绿（version-check 8/8、package-command-paths 30/30；原文「9/9、31/31」为估计值，31 为删前基数、删 1 后实为 30）。pi 根 CHANGELOG 已补 follow-up 条目对齐声称与实际。剩余：⑤ 待用户在 worktree 自行 `git add` + `/commit`（建议 add 全部改动：开发侧实现 + 本次测试修复 + CHANGELOG；`/commit` 在功能分支上会 push 分支并 enable PR #5 auto-merge，CI 绿即合并、`fixes #4` 自动关 Issue；2026-09-22 定 commit 由用户亲自执行，AI 不申请、不代行）。
  **进展补充**（2026-09-22 12:26）：⑤ 已执行（用户 commit f771fcac9 含实现 + 测试修复 + CHANGELOG，已 push），但 CI（run 35685064701）Check 步骤过了（3 个 TS2493 修复生效）、Test 步骤红在 `model-registry.test.ts` 3 条用例——与本 PR 改动无关的基线漂移：`npm run build` 时 `generate-models` 联网刷新内置模型目录，上游目录已淘汰裸 `anthropic/claude-opus-4`（只剩 4.1+），测试断言引用的内置模型 ID 过期（main 基线 CI 当时绿只因拉到的目录尚含该 ID；upstream 亦以 eaf72ed4d 修同一批过期断言）。处置：本地全新 `npm ci` + `npm run build` 完整复现 CI 环境后，6 处 `"anthropic/claude-opus-4"` 改 `"anthropic/claude-opus-4.1"`（含 modelOverrides key，带标记写入；行为断言不变，只换目录内存在的模型 ID），单文件 85/85 绿、`npm run check` 过、连同 issue #4 三文件共 124/124 绿，pi CHANGELOG 已补 follow-up。剩余：待用户再次 `git add packages/coding-agent/test/model-registry.test.ts CHANGELOG.md` + `/commit`（push 后 CI 绿即 auto-merge、`fixes #4` 关 Issue）。
  **关联**：T4 同一条出题动线；PR 合并全绿后 T4、T5 一并归档。

## 🟡 黄色紧急度

- [ ] **T3** 处置本项目 `cases/` 存量与各项目 `test-cases/` 存量（记录：2026-09-21 21:02）
  **背景**：dev-workflow 2026-09-21 三次修订取消了「权威源 cases/ 仓库 → 项目 test-cases/ 镜像」单向分发机制与 pending/passed 目录状态机（项目仓库成为唯一记录），本项目 `cases/zcode-cli/pending/` 还压着两个未完成需求组（custom-env-rename、model-picker-env-prefix），各存量项目可能也有 `test-cases/` 旧目录——不处置的话，权威源已废、镜像语义已变，两边都会误导后续工作。
  **要做什么**：① 本项目 `cases/zcode-cli/pending/` 两组：与用户确认旧需求是否仍要做——仍要做则到 zcode-cli 仓库开 Issue 承接（正文可用原 requirement.md 内容）、用例按新体系重出（正式测试位置 + 先红后绿），不做则记录后清理；② 摸底各产软件项目的 `test-cases/` 存量：`passed/` 用例按项目测试框架搬进正式测试位置（带 `# TEST_CASES_WRITE_OK` 标记整组搬移、适配 import 路径），迁移经 PR + CI 验证全绿后删旧目录；`pending/` 未完成组同①处置；③ 全部处置完后删除本项目 `cases/` 目录（机制已取消，留着是误导），CHANGELOG 记录收尾。

## 🟢 绿色紧急度

- [ ] **T4** 为 pi fork 的 Issue #4 出测试（先红），写入 Worktree `~/Developer/pi-fork-update-check`（记录：2026-09-21 21:08；更新：2026-09-22 11:00）
  **背景**：用户在 pi fork（xhqing/pi）上提了 Issue #4——fork 之后 update check 与 pi update 仍指向 upstream `pi.dev`，要求修复前先出测试。按 dev-workflow 测试先行流程（2026-09-21 修订版），测试 Agent 在功能分支上先出题、自跑见红，开发 agent 才开工实现到绿。（本条曾于 21:01 以 T3 编号记录，后被另一会话的并发 TODO 重写覆盖丢失，无归档痕迹，21:08 按新流程重写恢复并顺延编号为 T4；T3 现由「处置 cases/ 存量」占用。）
  **要做什么**：① 读 Issue #4 正文与评论，弄清预期行为（update check 指向哪、更新命令从哪拉取版本）；含糊处在 Issue 评论里与用户澄清，不猜——Issue 正文就是需求文档与验收标准，不再另写 requirement.md；② 用 pi 项目现有测试框架在项目正式测试位置（项目测试目录惯例位置）写测试，写入带 `# TEST_CASES_WRITE_OK` 授权标记；③ 自跑确认红（实现尚不存在，断言必红——验证断言有效，防永真断言）；④ 经用户明确授权后 commit 进该功能分支。不写 cases/ 权威源、不建 test-cases/ 目录——该体系已于 2026-09-21 随 Issue 状态机一并取消，项目仓库就是唯一记录（Issue + 测试都在 GitHub 上）。
  **进展**：①～③ 已完成（2026-09-21 21:50）——分支 `fix/fork-update-check` 上新增 `test/fork-version.test.ts`（根 VERSION 与 package.json/运行时 VERSION 一致）、改 `test/version-check.test.ts`（检查源指向 xhqing/pi 的 GitHub Releases、tag_name 协议，删 packageName/note 透传用例）、改 `test/package-command-paths.test.ts`（新增 managed/npm 两路径零 pi.dev 请求用例，二选一断言兼容「指向 fork 源」或「禁用+指引」两种修法，删 renamed-package 两用例），CHANGELOG.md 已记；自跑 13 红 27 绿，红全为 Issue 验收点与协议传导。剩余：④ 待用户授权 commit。
  **进展补充**（2026-09-22 11:00，开发侧会话复盘）：④ commit 已完成（698323171，2026-09-21 22:06 提交并 push，PR #5 建立，带 `fixes #4`）；但复盘发现该提交有四处执行遗漏（上述「删 renamed-package 两用例」实删一条等，完整清单见 T5）——PR #5 的 CI 在 Check 步骤红，①～③ 的成果里混着 2 条漏改旧用例与 3 个类型错误。修复由 **T5** 跟踪，本条与 T5 在 PR 合并全绿后一并归档。
