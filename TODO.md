# TODO（Hopper · 软件测试工程师）

> 活跃待办清单。条目按紧急度分节，节内按记录时间排序。处理完毕移入 `TODO-archive.md`。
> 个人隐私类待办写 `TODO.local.md`（不入 git），编号与公开版共用同一套空间。

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

- [ ] **T7** 为 ghostty-launcher（Atlas 项目）Issue #5 出测试（先红：面板 New Window 新窗口未置前），写入 worktree `~/Developer/ghostty-launcher-new-window-focus`（记录：2026-09-30 19:58）
  **背景**：用户 2026-09-30 在 ghostty-launcher **v0.2.1 正式版**实测报缺陷——点面板 **New Window** 没有正常切换窗口：新窗口**确实创建了**（Output 日志与窗口列表都能看到它），但**没有切到最上层**（Ghostty 在后台时，新窗口停在原前台应用后面）。根因已由 Atlas 源码级定位（ghostty 分叉仓库）：AppleScript `new window` handler（`macos/Sources/Features/AppleScript/AppDelegate+AppleScript.swift:171` → `TerminalController.newWindow(...)`，其内部 `TerminalController.swift:305` 只有 `DispatchQueue.main.async { c.showWindow(self) }`）**全程没有 `NSApp.activate`**；而 `activate window`（`ScriptWindow.swift:178`）做的正是 `makeKeyAndOrderFront` + `NSApp.activate(ignoringOtherApps: true)`——面板「点窗口行激活」走的就是这条、实测有效。修复触及 `lib/ghostty.js` 的 JXA 脚本生成与 `runNewWindow` 判定，属核心开发，按 dev-workflow 测试先行。
  **要做什么**（worktree `~/Developer/ghostty-launcher-new-window-focus` / 分支 `fix/new-window-focus`，测试文件写入带 `# TEST_CASES_WRITE_OK`）：① 读 Issue #5（https://github.com/xhqing/ghostty-launcher/issues/5）正文（含复现、期望行为、修复方向）；② 在项目正式测试位置 `test/` 出测试，覆盖三件事——（a）`buildNewWindowScript` 生成的脚本在 `newWindow(...)` 之后调用 `activateWindow`（同时用 JSON 转义后的目录 / 登录 shell 断言既有契约不回归）；（b）`runNewWindow` 对「窗口已创建但激活失败」仍返回 `ok:true`（**不得因激活失败判失败去重试，否则会建出第二个窗口**），且该情况写进 `log`（可观测、不静默）；（c）创建失败（脚本报错）仍走「重试一次 → `{ok:false,error}`」且**绝不回退 `open -na`**（既有断言保持绿）；③ 自跑确认红（实现尚不存在）；④ 测试命令沿用 `npm test` / `npm run check`；⑤ 交付自检全过后报告测试文件清单 / 测试命令 / 自跑见红证据——**测试产物留在工作区不提交**（2026-09-30 用户定新规矩：测试与实现在开发完成后同一次提交、同一个 PR，不单独提交、不单独开 PR）。
  **关联**：Issue #5（ghostty-launcher）；开发侧 Atlas 已建 worktree + 分支 `fix/new-window-focus`（从最新 main `eb22957` 开出，2026-09-30）；本项目 T6（Issue #1 出题）的后续。测试就位后 Atlas 实现到绿 → 本地全量绿 → 用户一次性 `git add`（测试 + 实现 + CHANGELOG）+ `/commit` → PR（`fixes #5`）→ CI 绿 auto-merge → 合并后 `/bump` + `/release` 发新版。
  **进展**（2026-09-30 20:08）：①～⑤ 已完成——`test/ghostty.test.js` 新增 6 条用例（文件 24 条、全仓 31 条），并定下协议：脚本在窗口创建成功但激活失败时返回 `"activate-failed"`（与创建失败区分），`runNewWindow` 收到它按成功处理（`ok:true`）、不重试、不回退 `open -na`、写警告日志。覆盖：（a）脚本在 `newWindow` 之后对刚创建的窗口对象调用 `activateWindow` + JSON 转义既有契约；（b）在 Node 沙箱真实执行脚本断言控制流——激活抛错返回 `"activate-failed"`、创建抛错不得伪装成成功；（c）`runNewWindow` 的激活失败成功判定与警告日志；（d）非约定状态仍按创建失败重试报错（不放宽失败判定）。自跑：`npm run check` 全过、`npm test` 4 红 27 绿（红全为新增 Issue #5 行为用例、实现尚不存在；2 条新守卫用例 + 既有 18 条保持绿）。自检：`tmp/` 沙箱按 Issue 修复方向改 lib 副本对跑 24/24 全绿，四类变异（脚本不激活 / 创建失败吞成激活失败 / 不识别 `activate-failed` / 放宽成功判定）分别红 3/1/1/1 条，均被对应用例抓住。ghostty-launcher `CHANGELOG.md` 已加「未发布」节测试条目；测试文件以未提交工作区改动留在 worktree（不 `git add`、不提交）。剩余：Atlas 实现到绿 → 本地全量绿 → 用户一次性 `git add`（测试 + 实现 + CHANGELOG）+ `/commit` → PR（`fixes #5`）→ CI 绿 auto-merge → `/bump` + `/release` 发新版；全流程完成后本条归档。

- [ ] **T8** 为 ghostty-launcher（Atlas 项目）Issue #8 出测试（先红：置前结果实测 + LaunchServices 兜底 + 失败可见），写入 worktree `~/Developer/ghostty-launcher-front`（记录：2026-10-09 14:37）
  **背景**：用户 2026-10-09 实测报缺陷——在 VSCode 里点面板 **New Window**，界面上什么都没发生（没弹窗、也没切前台），但扩展日志写着 `✓ 新窗口创建成功`、窗口列表计数 +1（窗口确实建出来了）。Atlas 排查锁定根因：置前请求在 AppKit 之下被 macOS 否决——系统日志（WindowServer 子系统）同一瞬间记录 `CPS: Rejecting expired request`（判定式：请求来源时间 T1 = 该应用最后一次「人操作过它」的时间，早于 T2 = 该应用最近一次被切到前台的时间即拒），失败那次 15 条拒绝逐条对应 12 次点击；同日再试（人刚操作过 Ghostty，T1 晚于 T2）同一路径被放行、窗口正常置前——即**条件性否决**、表现为偶发不置前。而扩展用的 `activate window` 底层 `NSApp.activate(ignoringOtherApps:)` **异步无返回值**，AppleScript 只能返回「请求已发出」，拿不到系统拒收结果，所以日志全绿而实际没置前。这不是 Issue #5 修复的回归，而是它的覆盖缺口（0.2.2 验证时 Ghostty 恰好在前台）。修复触及 `lib/ghostty.js` 与 `extension.js` 的置前判定，属核心开发，按 dev-workflow 测试先行。
  **要做什么**（worktree `~/Developer/ghostty-launcher-front` / 分支 `fix/bring-to-front`，测试文件写入带 `# TEST_CASES_WRITE_OK`）：① 读 Issue #8（https://github.com/xhqing/ghostty-launcher/issues/8）正文（现象、复现、根因、期望行为/验收标准）；② 在项目正式测试位置 `test/` 出测试，**按下面已定的接口契约**覆盖新增纯逻辑（`test/ghostty.test.js` 追加即可，既有 31 条必须保持绿）；③ 自跑确认红（实现尚不存在——新增函数取到就是 `undefined`，断言必红）；④ 测试命令沿用 `npm test`（`node --test test/*.test.js`）与 `npm run check`；⑤ 交付自检全过后报告测试文件清单 / 测试命令 / 自跑见红证据——**测试产物留在工作区不提交**（测试与实现同一次提交、同一个 PR）。
  **约定接口（开发侧已定稿，测试按此断言）**：`lib/ghostty.js` 新增两个导出——（1）`ensureForeground({ isFrontmost, activateApp, log = noop, wait = noop }) → Promise<{ ok, method, error? }>`：已在前台 → `{ ok:true, method:'none' }` 且**绝不调用** `activateApp`（不多余动作）；不在前台 → 调用 `activateApp()` 一次做 LaunchServices 兜底（真机即 `open -a Ghostty.app`）→ 复查，复查在前台 → `{ ok:true, method:'launchservices' }`；兜底后仍不在前台 → `{ ok:false, method:'launchservices', error }`（error 是一句可直接给用户看的中文原因）；`isFrontmost()` 抛错或 `activateApp()` 抛错都**不得向上抛**（不 reject）——前者记日志后按「未能确认」继续走兜底、以复查结果为准，后者返回 `ok:false` + error；`wait()` 在每次查询前调用（注入延时，测试注入 noop；后台路径 ≥2 次、已在前台路径 ≥1 次）；成功 / 兜底 / 失败各写一条可辨识日志（不静默）。（2）`foregroundNoticeText({ created, error }) → string`：`created:true` 时含「新窗口已创建」「未能切到前台」，`created:false` 时含「未能切到前台」，两者都含操作指引「⌘-Tab」，并包含传入的 error 文本——供面板提示与状态栏路径复用。
  **不要做的事**：不改 `runNewWindow` / `runActivate` / `buildNewWindowScript` 的既有契约（Issue #5 锁定的行为——创建成功后 `ok:true`、失败只重试一次、绝不回退 `open -na`——全部保持绿）；不真调系统（`open` / `osascript` 一律注入假实现）。
  **关联**：Issue #8（ghostty-launcher）；worktree `~/Developer/ghostty-launcher-front` / 分支 `fix/bring-to-front`（从最新 main `5304e22` 开出，2026-10-09）；本项目 T7（Issue #5 出题）的后续。测试就位后 Atlas 实现到绿 → 本地全量绿 → `/add` → `/commit` → PR（`fixes #8`）→ CI 绿 auto-merge → 用户试用预发布 → 满意后发版。
  **进展**（2026-10-09 14:47）：①～⑤ 已完成——`test/ghostty.test.js` 新增 10 条用例（文件 34 条、全仓 41 条），按 T8 约定接口覆盖：（a）`ensureForeground`——已在前台 `method:'none'` 且不调 `activateApp`；不在前台兜底恰一次、复查在前台 `method:'launchservices'`；兜底后仍不在前台 `ok:false` + 中文原因；查询抛错 / 复查抛错 / 兜底动作抛错均不 reject；`wait` 每次查询前调用（后台 ≥2、已在前台 ≥1）；成功 / 兜底 / 失败日志可辨识。（b）`foregroundNoticeText`——`created:true` 含「新窗口已创建」，两分支均含「未能切到前台」「⌘-Tab」与传入原因；`created:false` 不得声称已创建；error 缺省不抛错。自跑：`npm run check` 全过；`npm test` 10 红 31 绿（红全为新增 Issue #8 用例、实现尚不存在——7 条直接报 `is not a function`，3 条 assert.fail 捕获到同一 TypeError），既有 31 条保持绿。自检：`tmp/` 沙箱按接口契约写参考实现拼接 lib 副本对跑 34/34 全绿（排除永真断言），四类变异（已在前台也调兜底 / 兜底失败谎报成功 / 查询异常向上抛 / 文案误称窗口已创建）各自红 1 条、均被对应用例抓住。测试文件走 `# TEST_CASES_WRITE_OK` 标记通道写入（`tmp/` 预置 → 拼接 → 带标记 cp；diff 确认既有 401 行逐字节未变、208 行纯新增）。ghostty-launcher `CHANGELOG.md` 已加「未发布」节测试条目；测试文件以未提交工作区改动留在 worktree（不 `git add`、不提交）。剩余：Atlas 实现到绿 → 本地全量绿 → `/add` → `/commit` → PR（`fixes #8`）→ CI 绿 auto-merge → 用户试用预发布 → 满意后发版；全流程完成后本条归档。

- [ ] **T22** 为 XPilot（Atlas 项目）Issue #5 + #6 出测试（先红：陈旧实例自动恢复 + 遗留 /tmp 清理 + 定时订阅同步），写入 worktree `~/Developer/XPilot-subscription-sync`（记录：2026-10-27 12:11）
  **背景**：XPilot 0.6.4（PR #3，2026-10-26）修掉了「静默断网」的**报告**侧——运行状态迁出 `/tmp`、`status` 用 PID + 端口 + 进程名交叉校验、`start` 幂等、配置陈旧时黄字提示 `xpilot restart`。但「发现之后自动处理」仍是空的，Issue #5 与 #6 就是这两块缺口，用户 2026-10-27 要求合并成一轮解决：
  - **Issue #5**（https://github.com/xhqing/XPilot/issues/5，OPEN）：① 运行实例的配置与当前默认节点不一致时，`status` 只提示、无人动手重启——2026-10 事故正是节点 IP 漂移、跑着的 xray 咬内存里的旧地址、代理「在线」却全部境外连接超时；要求 monitor 自动重启一次并记原因，且**不能重启死循环**（重启修不好的情况有上限后退回只告警）。② 0.6.3 及更早版本遗留在 `/tmp` 的 `xpilot.pid` / `xpilot-xray.json` / `xpilot-xray-*.log` / `xpilot-auto-switch.pid` 新代码不读、是惰性的，但事后排查事故时极具误导性；要求 `start` / `stop` 时确认无对应进程存活后清掉并记日志。
  - **Issue #6**（https://github.com/xhqing/XPilot/issues/6，OPEN）：`settings.json` 里 `subscription.auto_update` / `subscription.update_interval` 是**死配置**——`config.py` 生成默认值、两版 README 都列着，但全仓库无人读取（唯一真正生效的是 `auto_switch.auto_update_subscription`，语义是「全节点挂掉时的应急刷新」，不是定时同步）。用户已选**方案 A（实现它）**：monitor 按 `update_interval` 定时拉订阅，与 Issue #5 的自动重启配套（拉下来不生效等于白拉）。

  **要做什么**（worktree `~/Developer/XPilot-subscription-sync` / 分支 `feat/subscription-sync-and-recovery`，从最新 main `9ae2d8d` 开出，测试文件写入带 `# TEST_CASES_WRITE_OK`）：① 读 Issue #5 / #6 正文；② 按下面**已定稿的接口契约**在项目正式测试位置 `tests/` 出测试（建议新建 `tests/test_legacy_cleanup.py` / `tests/test_auto_recover.py` / `tests/test_subscription_sync.py`，也可按判断合并；既有 160 条必须保持绿）；③ 自跑确认红（新函数尚不存在——`AttributeError` / `ModuleNotFoundError` / `ImportError` 即红，断言必红）；④ 测试命令沿用 `python3 -m pytest -q`（项目**没有** `python` 别名，必须 `python3`；静态检查 `ruff check xpilot tests --select E9,F63,F7,F82`）；⑤ 交付自检全过后报告测试文件清单 / 测试命令 / 自跑见红证据——**测试产物留在工作区不提交**（测试与实现同一次提交、同一个 PR，不 `git add` / 不 commit / 不 push / 不开 PR）。

  **约定接口（开发侧已定稿，测试按此断言；实现尚未写入，契约即验收口径）**：

  **（1）新模块 `xpilot/legacy_cleanup.py`**（Issue #5 第 2 条）
  ```python
  LEGACY_PID_FILES = ('xpilot.pid', 'xpilot-auto-switch.pid')
  LEGACY_FILES = ('xpilot-xray.json', 'xpilot.log')
  LEGACY_LOG_GLOBS = ('xpilot-xray-*.log',)

  def find_legacy_paths(tmp_dir: str = '/tmp') -> list[str]: ...
  def cleanup_legacy_state(tmp_dir: str = '/tmp', *, pid_alive=None, log=None) -> list[str]: ...
  ```
  行为约定：
  - `find_legacy_paths`：返回 `tmp_dir` 下**实际存在**的遗留文件绝对路径；glob 按 `LEGACY_LOG_GLOBS` 展开；去重；按完整路径字符串升序排序（保证断言稳定）；不存在的不返回（空列表）。
  - `cleanup_legacy_state`：**返回实际删除的路径列表**（顺序同 `find_legacy_paths`）；从未删除成功的文件不得出现在返回值里。
  - **PID 文件特殊处理**：读取文件内容为整数 PID；`pid_alive(pid)` 返回真 → **跳过不删**、记一条 warning（内容须含该路径，便于断言）；`pid_alive` 返回假 → 正常删。文件内容不是合法整数 / 为空 / 读取失败 → 视为无活进程，照删。
  - `pid_alive` 默认实现：`os.kill(pid, 0)`——`ProcessLookupError` → `False`，`PermissionError` → `True`（无权信号说明进程存在），其它 `OSError` → `False`。测试**必须注入** `pid_alive`，不得依赖真实进程。
  - 删除单个文件抛 `OSError` → 记一条 warning（含路径）、继续处理其余文件、**不向上抛**、不计入返回值。
  - `log` 默认实现转发到模块 logger，**签名固定为 `log(level: str, message: str)`**（`level` ∈ `{'info', 'warning'}`）；测试注入收集器断言内容。
  - 幂等：第二次调用返回空列表、不报错。
  - 全部 `tmp_dir` 可注入是硬要求——测试用 `tmp_path` 造文件，**绝不触碰真 `/tmp`**。

  **（2）`ProxyManager` 新增方法 `cleanup_legacy_state(self) -> list[str]`**（Issue #5 第 2 条接入）
  - 委托 `xpilot.legacy_cleanup.cleanup_legacy_state(log=...)`，`log` 转发到 `ProxyManager` 模块 logger；返回删除列表。
  - `start()` **进入后第一步**调用它；`stop()` **返回前**调用它（`restart()` 复用两者，天然覆盖）。
  - 断言方式建议：monkeypatch `xpilot.proxy_manager.cleanup_legacy_state`（或模块内已导入的符号）为 spy，验证 `start` / `stop` 各调用一次；或直接注入假实现验证调用。

  **（3）`AutoSwitch` 新增状态与配置读取**（`src` 为 `xpilot/auto_switch.py`）
  `start()` 里从 `settings.json` 读两块配置并存为实例属性（**旧配置缺块时必须用默认值，不能抛错**）：
  ```python
  "subscription": {"auto_update": true, "update_interval": 3600},
  "auto_recover": {"enabled": true, "max_restarts": 3, "restart_window": 1800,
                    "min_interval": 60, "dead_restart_ticks": 2}
  ```
  属性名固定：`_subscription_sync_enabled`（bool）/ `_subscription_sync_interval`（int）/ `_auto_recover_enabled`（bool）/ `_recover_max_restarts`（int）/ `_recover_window`（int）/ `_recover_min_interval`（int）/ `_dead_restart_ticks`（int）。新增计数器：`_last_sub_sync`（float）/ `_recover_times`（`list[float]`，窗口内重启时间戳）/ `_dead_ticks`（int）。
  - `update_interval` 非法（`<=0` / 非数字）→ 回退 `3600`；`dead_restart_ticks <= 0` → 视为**禁用**全挂重启（`_track_dead_nodes` 恒返回 `False`）。
  - `start()` 结束时 `self._last_sub_sync = time.time()`（首次定时同步在**一个 interval 之后**，避免每次 `xpilot start` 都拉订阅）。
  - `_tick_interval`：已启用子系统的间隔最小值再与 `30` 取 min（即 tick **不超过** 30 秒——保证陈旧检查粒度）；四个子系统（watchdog / auto_switch / subscription_sync / auto_recover）**全禁用**时才不启动 monitor。

  **（4）`AutoSwitch` 新增五个方法（签名与语义固定）**
  ```python
  def _refresh_all_subscriptions(self) -> int: ...
  def _subscription_sync_tick(self) -> bool: ...
  def _recover_restart(self, reason: str) -> bool: ...
  def _check_stale_runtime(self) -> bool: ...
  def _track_dead_nodes(self, usable_count: int) -> bool: ...
  ```
  - **`_refresh_all_subscriptions()`**：读 `settings.json` 的 `subscriptions`（dict，名称 → URL），逐个调 `self.node_manager.import_from_subscription(url, update_existing=True, source=name)`；累加返回的节点数；单个源抛异常只 `logger.error` 并继续下一个；无订阅源 → 返回 `0`（不抛）。
  - **`_subscription_sync_tick()`**（Issue #6 定时同步）：`not self._subscription_sync_enabled` → `False`；距 `_last_sub_sync` 不足 `_subscription_sync_interval` → `False`；否则**先更新** `_last_sub_sync = now`（失败也不高频重试）→ 调 `_refresh_all_subscriptions()` → 结果 `>0` 记 info 并返回 `True`，否则返回 `False`。
  - **`_recover_restart(reason)`**（统一重启执行器）：① `not _auto_recover_enabled` → `False`；② `not self.proxy_manager.is_running()` → `False`（不负责拉起，watchdog 的活）；③ 裁剪 `_recover_times` 到 `restart_window` 内；④ 窗口内次数 `>= _recover_max_restarts` → 记 warning（含「上限」字样与 reason）、返回 `False`（**退回只告警**）；⑤ `_recover_times` 非空且距最后一次不足 `_recover_min_interval` → `False`（防抖）；⑥ 调 `self.proxy_manager.restart()`，异常捕获记 error、**不抛出**；⑦ 追加 `now` 到 `_recover_times`、记 info（含 reason）；⑧ 返回 `True`。
  - **`_check_stale_runtime()`**（Issue #5 第 1 条）：`not self.proxy_manager.is_running()` → `False`；`matched, detail = self.proxy_manager.running_config_matches_current()`；`matched` 为 `True` 或 `None`（无法判定）→ `False`（**只处理明确的 `False`**）；`False` → `return self._recover_restart(f'运行实例配置陈旧（{detail}）')`。
  - **`_track_dead_nodes(usable_count)`**（Issue #5 第 1 条的另一触发路径）：`usable_count > 0` → `_dead_ticks = 0` 并返回 `False`；否则 `_dead_ticks += 1`；`_dead_restart_ticks <= 0` 或 `_dead_ticks < _dead_restart_ticks` → `False`；达到阈值 → **先把 `_dead_ticks` 归零**、再 `return self._recover_restart(f'连续 {n} 轮全部节点真实流量不通')`。

  **（5）既有方法的接入点（断言据此）**
  - `_check_and_switch()`：`if not usable:` 分支里，应急刷新后仍无可用节点时，先 `self._track_dead_nodes(0)` 再 `return`；拿到 `usable` 后、选最优前调一次 `self._track_dead_nodes(len(usable))`（用于归零）。
  - `_monitor_loop()`：每个 tick 若 `_auto_recover_enabled` 就调 `self._check_stale_runtime()`（**每 tick 都查**，不受其他子系统间隔约束）；每 tick 若 `_subscription_sync_enabled` 就调 `self._subscription_sync_tick()`（间隔由其内部判定）。

  **不要做的事**：① 不改 `running_config_matches_current()` 的既有契约（`True` / `False` / `None` 三态语义与返回的 `(matched, detail)` 元组）；② 不改 `_try_refresh_subscription()` 的既有语义（应急刷新、`auto_update_subscription` 开关、独立 `subscription_refresh_cooldown` 退避）——它**不是**定时同步，两条路径并存互补；③ 不动 `probe_runtime()` / `start()` 幂等 / 多实例拒绝这些 0.6.4 已锁定的行为（既有 44 条 Issue #1 用例必须保持绿）；④ 不真调 `xray` / 系统命令 / 真实网络（`subprocess` / `requests` 全注入假实现）；⑤ 测试不得读写真实 `/tmp` 下的 xpilot 文件。

  **关联**：Issue #5 + Issue #6（`xhqing/XPilot`，均 OPEN，PR 描述将带 `fixes #5` 与 `fixes #6`）；worktree `~/Developer/XPilot-subscription-sync` / 分支 `feat/subscription-sync-and-recovery`（从最新 main `9ae2d8d` 开出，2026-10-27）；本项目 T21（Issue #1 出题，已闭环归档）的后续。测试就位后 Atlas 实现到绿 → 本地全量绿 → 用户 `/add` + `/commit` → PR → CI 绿 auto-merge → 自动发版链；全流程完成后本条归档。

  **进展**（2026-10-27 12:16）：出题完成，已交 Atlas（测试产物留在 worktree 未提交）。新增 3 个测试文件共 70 条用例——`tests/test_legacy_cleanup.py`（19 条：名单常量、`find_legacy_paths` 排序/空集/glob 展开、删除返回值、活 PID 跳过并警告、非法内容照删、默认 `os.kill` 语义、单个删除失败不断链、模块 logger 转发、幂等，以及 `ProxyManager` 委托与 `start`（探测前）/ `stop`（返回前）/ `restart`（两次）接入）、`tests/test_auto_recover.py`（26 条：配置块与缺块默认值、四个子系统的 tick 间隔上限 30s、`_recover_restart` 的禁用/未运行/限次/窗口裁剪/防抖/异常不抛、`_check_stale_runtime` 三态（只处理 `False`）、`_track_dead_nodes` 阈值与先归零、`_monitor_loop` 每 tick 接线）、`tests/test_subscription_sync.py`（25 条：`subscription` 配置与非法间隔回退、`_last_sub_sync` 起点与「先推进后刷新」、仅订阅子系统启用也能拉起 monitor、`_refresh_all_subscriptions` 逐源与单源失败隔离、`_subscription_sync_tick` 间隔/失败不收紧重试、`_check_and_switch` 的全挂与恢复轮次喂给 `_track_dead_nodes`）。自跑见红：`python3 -m pytest -q --continue-on-collection-errors` → 46 failed / 165 passed / 1 collection error（`test_legacy_cleanup.py` 因 `xpilot.legacy_cleanup` 不存在整模块 ImportError；另两个文件红的类型是 `AttributeError: 'AutoSwitch' object has no attribute ...`；绿的是 160 条既有用例 + 5 条「旧行为本来就成立」的负向守卫）；既有 160 条单独跑全绿。防永真断言：`tmp/t22/sandbox/` 按契约写参考实现后 230 条全绿，另做 12 组变异（start 不清理、PID 存活照删、删除失败上抛、`None` 当陈旧、去掉防抖、去掉窗口上限、先重启后归零、失败也推进时间戳、刷新后才推进时间戳、间隔不生效、tick 不封顶、可用轮次不喂 `_track_dead_nodes`）逐一被对应用例抓住。`ruff check xpilot tests --select E9,F63,F7,F82` 通过。待 Atlas 实现到绿 → 同一次提交 + 同一个 PR（`fixes #5` / `fixes #6`）。

  **进展补充**（2026-10-27 12:2x）：Atlas 实现已落地、全量 230 条绿；按其实现决定补了 4 组断言（共 12 条）——`tests/test_config.py` 新增 `TestSubscriptionFlagMigration` 8 条（旧默认值常量、全等则迁移、三种全等以外不动、块缺失不动、无文件不创建、JSON 损坏不覆盖）、`tests/test_auto_recover.py` 新增 4 条（重启失败返回 False 且占配额、连续失败到上限后不再调用 restart、`dead_restart_ticks<=0` 不积累 `_dead_ticks` 参数化 2 条）、`tests/test_subscription_sync.py` 将缺块默认值断言由「bool 类型」收紧为 `is True`。自跑 **242 passed**、ruff 通过、`test_config.py` 的 diff 为纯新增 81 行（既有用例零改动）；5 组变异（迁移不执行、迁移判据写反、失败不占配额、禁用仍累加、缺块默认改回 false）全部被抓住。**发现一个判据层面的遗留问题已报 Atlas**：新默认间隔也是 3600，用户把 `auto_update` 改为 `false` 而间隔保持 3600 时，得到的块恰好等于旧默认值，下次任何 `xpilot` 命令都会再迁回 `true`（退出只能靠同时改间隔）；建议改用一次性迁移标记或「无 `auto_recover` 块才迁移」的判据，等 Atlas 定方案后补断言。


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


