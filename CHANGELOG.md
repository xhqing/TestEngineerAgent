# CHANGELOG

本文件记录本项目（TestEngineerAgent / Hopper）每次文件增删改查的变更，写清「为什么改」和「改了什么」。版本号以项目根 `VERSION` 文件为唯一权威（当前 0.1.0）。

## [Unreleased]

### 变更（CLAUDE.md 删去「由 Claude Code 自动加载」说明句）

- **为什么改**：用户 2026-09-12 要求 CLAUDE.md 不再强调本文由 Claude Code 加载，团队全部项目的 CLAUDE.md 统一清理此类语句。
- **改了什么**（2026-09-12）：`CLAUDE.md` 开头角色定位行删去句尾「本文件由 Claude Code 在每次会话开头自动加载。」，角色描述本身保留。

### 变更（通用能力句式去 find-skill 提及）

- **为什么改**：全局 find-skill skill 已被用户删除（实际使用中从未用到），各处不再提及；本项目 CLAUDE.md 通用能力句式仍列着 find-skill，2026-09-12 联动清理。
- **改了什么**：`CLAUDE.md` 通用能力句式「（anysearch 实时搜索、find-skill 找 skill 等）」→「（anysearch 实时搜索等）」。

### 变更（assets/logo.svg 副标题去中文）

- **为什么改**：全局规则新增「Logo / 图标资产文字一律用英文」（2026-09-12 用户立，起因 Swing 仓库 logo 副标题混入中文被指出）：logo 是面向全球读者的视觉标识，中文受众已有 README_cn.md 双语通道；且 SVG 中文依赖查看环境的字体回退，渲染不可控。本次为按新规批量清理存量。
- **改了什么**：`assets/logo.svg` 副标题「Test Engineer · 软件测试工程师」→「Test Engineer」。

## [0.1.0] - 2026-09-06

### 新增（第二个需求组：zcode-cli 配置文件改名 custom-provider.env → custom.env，2026-09-08）

- **为什么改**：用户指令——按文件命名规范（删冗余修饰词）把自定义供应商配置文件 `custom-provider.env` 简化为 `custom.env`。全仓摸底：代码 7 文件 / 测试 5 文件 / 文档与模板若干、共 41 处引用，文件名唯一权威源是 `env-config.ts` 的 `customProviderEnvFileName` 常量；用户本机实际存在该文件，存量自动迁移是刚需（项目已有 `.env` → `custom-provider.env` 的一次性 rename 先例可循）。
- **改了什么**：新建权威源 `cases/zcode-cli/pending/custom-env-rename/`——`requirement.md`（五条验收条目 R1–R5：默认路径改 custom.env、两层 legacy 自动迁移（custom-provider.env / .env → custom.env，ZCODE_ENV_FILE 覆盖时跳过）、新旧并存不丢数据（新名生效旧文件保留）、模板与文档全量同步、解析与同步语义零变化；边界注明 config.json 的 env-<provider> 槽位名不变、与 model-picker-env-prefix 组互不阻塞）+ 验收用例 `custom-env-rename.test.ts`（bun:test 5 条，临时 HOME fixture 直测 envFilePath / migrateLegacyEnvFile）。同步进 zcode-cli `test-cases/pending/custom-env-rename/`（带标记），diff 校验一致。验证：5 条 3 红 2 绿（红 = R1、R2×2 待开发契约；绿 = ZCODE_ENV_FILE 既有行为锁定 + 并存终态契约锁定），符合 pending 预期。

### 新增（首个需求组：zcode-cli /model 与 /settings 选择器去 env- 内部槽位前缀，2026-09-08）

- **为什么改**：用户提需求——TUI 输入 `/model` 后不希望看到带 `env-` 前缀的选项。调研定位根因：custom-provider.env 配置同步进 config.json 时落在内部槽位 `env-<provider>`，`/model` 平铺列表中 env 独有条目（官方槽位没有的同名模型）以原始内部 id 显示（如 `env-zai/glm-4.7`），`/settings → Model providers` 级联中同一供应商被拆成官方 + env 两个重复组；而代码既有设计意图（`displayProviderId` 注释）本就是「env- 前缀用户永远不该看到」，属显示层未清理遗留。两个模糊点经用户拍板：处理方式取「去掉前缀显示、条目保留可选」（非隐藏条目——只用 custom-provider.env 的用户仍需看到自己配置的模型）；范围取「/model 与 /settings 两处一并修齐」（同一根因）。
- **改了什么**：新建权威源首个需求组 `cases/zcode-cli/pending/model-picker-env-prefix/`——`requirement.md` + 验收用例 `model-picker-env-prefix.test.ts`（bun:test，数据形态对齐 runtime listModels() 两种既有格式；含边界非目标——config.json 内部存储不动、既有孪生去重保持、给开发的注意点——`test/selectors.test.ts` 中锁定旧契约的 `keeps env-file slot entries without an official twin` 需随实现同步更新）。带 `# TEST_CASES_WRITE_OK` 标记同步进 zcode-cli `test-cases/pending/model-picker-env-prefix/`，diff 校验镜像与权威源逐字节一致。首版 6 条（R1–R5 锁新契约、R6 锁既有行为），全量 `bun test` 752 条自动拾起（原 746 + 新 6）、原有测试零回归。
- **同日增补（R7/R8 选择路径防护）**：用户反馈已发布的 v3.8.1-31 未登录选模型后连不上上游、被迫回退版本，要求用例确保此类问题不再漏网——首版用例只锁「列表显示层」，没锁「选中后解析到有凭证槽位」，而 3.8.1-31 事故恰是两层组合缺陷。用例组扩到 8 条：R7（登出态下去重后列表每个条目经 resolveModelSlotRef 必须落到带凭证 env 槽——3.8.1-31 事故场景的永久防护）、R8（去前缀 env 独有条目解析闭环），采用临时 HOME fixture 复刻未登录 config 形态；requirement.md 验收条目与人工门禁同步补齐（人工验收必须逐条选择确认实际能对话）。
- **同日二次增补（R9/R10 未登录过滤，需求升级）**：用户追加需求——未登录状态下 `/model` 不应显示需登录才能用的模型，只显示 custom-provider.env 配置的可用模型。需求组从「显示层去前缀」升级为「按登录状态过滤 + 去前缀」的行为契约变更。两个分叉经用户拍板：过滤范围取「/model 与 /settings 级联一并」（级联未登录时官方组还显示会诱导保存指向无凭证槽位的配置）、空列表取「显示提示」（未登录且无 custom-provider.env 可用模型时提示登录或配置，不静默）。变更内容：requirement.md 加 R9（未登录只显示 env 槽声明且带凭证的模型，登录态全量不变——官方模型登录后回到列表）、R10（空列表提示）；R7 随 R9 改写（未登录即使 runtime 混报官方条目，列表也只含 env 条目且全部解析闭环）；注意点补 selector 建议签名 `modelPicker / providerModelPicker(options, current, signedIn?: boolean)`（undefined 现状兼容）、TUI 三个调用点（showModelPicker / showModelProviderSettings / 快捷循环切换）用 readSignedInProvider 取登录态、空列表提示在 showModelPicker 层实现。用例组 8→11 条（R1–R5 显式传 signedIn、R9 两条：平铺未登录过滤 + 登录全量回归、级联未登录过滤；R10：仅官方条目时未登录 picker 为空）。重新同步镜像并验证：11 条 8 红 3 绿（红 = R1–R5、R9×2、R10 待开发契约；绿 = R6 既有行为 + R7/R8 防护锁定——R7 现状绿属过滤前后输出碰巧相同，终态契约锁定仍有效），形态符合 pending 预期。

### 变更（归档流程加 diff 前置校验：同步单向性的执法机制，2026-09-08）

- **为什么改**：用户质询「权威源到镜像是否严格单向、不是的话 diff 校验就没意义」——确认设计上严格单向（镜像端唯一合法写入口 = Hopper 带标记同步、无反向回流场景），但授权标记非身份凭证、单向目前只靠纪律维持（约定单向 ≠ 物理单向）。归档前 diff 校验的意义恰在于此：把「单向」从流程约定变成可验证、可执法的不变量——镜像与权威源不一致即可归因为异常（旁路偷改 / 流程错序 / 过渡期手工用例未补录），暴露后以权威源回正。
- **改了什么**：`CLAUDE.md` 工作流程第 6 步归档动线改为「先校验后动作」：diff 项目 `test-cases/` 与权威源**该项目子目录** `cases/<项目名>/`（非整个 `cases/`；排除 `__pycache__`、`.pytest_cache` 等运行缓存，只比 git 跟踪内容最稳）→ 不一致一律停止上报、处置由用户裁决（含过渡期手工用例补录也需用户知情）→ 一致才在 main 验证全绿、权威源 `git mv`、同步、用户 commit。同步条款写入全局 dev-workflow 的 `references/test-cases.md` 归档节（权威源 `~/.claude/skills/`，四端软链即时生效，Prometheus 镜像随下次同步）。

### 变更（工作流大转向对齐：PR / CI 门禁改本地门禁，新增用例生产线流程，2026-09-08）

- **为什么改**：2026-09-07 用户对 dev-workflow 大转向定稿——远端 PR / CI 门禁取消、裁决全部本地化（本地全量测试机器门禁 + 用户装测试包人工门禁 + Hopper 归档验收复核），全局 dev-workflow skill 当晚已重写为本地门禁版；但本项目 `CLAUDE.md` 仍停留在旧方案（「裁决权归远端 CI」「核心改动走 PR 等绿合并」「workflow 配置归你管」）——角色文件每次会话必载、比 skill 更强势，Hopper 按旧文件行事会与现行流程打架。另查明：用户向 Hopper 提需求不会触发 dev-workflow（该 skill 是开发方视角、触发词为开发动作），Hopper 的流程入口必须是角色文件本身，故用例生产线流程写入 `CLAUDE.md`。
- **改了什么**：`CLAUDE.md` 整篇更新——① 职责四条重写：需求转用例（明确 `requirement.md` 是用户验收清单）、存量补网（保留）、用例库维护（权威源 `cases/<项目名>/{passed,pending}/<需求名>/` 单向同步进项目 `test-cases/` 镜像，结构权威指针指向 dev-workflow 的 `references/test-cases.md`，删 GitHub Actions workflow 提法）、归档验收（新增：开发合并回 main 后 pending→passed 挪组 + 第三方复核）。② 新增「你的工作流程（用例生产线）」六步动线：接需求澄清 → 产出需求组进权威源 → 同步进项目（写入通道：四端 hook 拦 `test-cases/` 写操作，Bash 末尾带 `# TEST_CASES_WRITE_OK` 授权标记）→ 通知用户 `git add` + `/commit` → 开发期间问题对齐 → 归档验收。③ 三权分立更新：裁决权由「远端 CI」改为「本地双重门禁 + 归档验收」（2026-09-07 废止标注），删 PR / CI 门禁表述，补「main 必须始终绿（passed 区）」精确口径与「pending 红不算 main 红」的占位语义。④ 工具段删 CI 行、补「验收用例必须能被项目现有测试命令执行」。⑤ 约束段补「写 `test-cases/` 一律走授权标记通道，不用无标记方式绕过 hook」。

### 变更（工作原则「分支开发、PR 门禁」修订：main 不设分支保护、CI 门禁改纪律制，2026-09-07）

- **为什么改**：用户 2026-09-07 裁定——main push 不做限制（方便 main 对齐）、普通文件处理修改不走 dev-workflow、只有存在测试用例的软件开发项目才走 dev-workflow。原原则中「main 配分支保护（CI 不绿不许 merge、禁止直接 push）」与新政策直接冲突。
- **改了什么**：`CLAUDE.md` 两处——① 「分支开发、PR 门禁」条改为「分支开发、CI 纪律门禁」：PR 全量用例（旧 + 新）全绿才合并 main 由 dev-workflow 流程纪律保证（等 CI 完成绿灯后再合并），不依赖 GitHub 分支保护；main 不设分支保护、允许直接 push，bump、文档等不新增测试用例、不碰核心功能的杂事直接在 main 上改、`/commit` 直推。② 「结果的裁决权归 CI」条补一句：main 允许直接 push 但直接 push 同样触发 CI——红灯事后暴露而非事前拦截，核心改动仍走 PR 等绿合并。关联的全局 dev-workflow / commit / bump / release skill 修订与 GitHub 分支保护撤除（zcode-cli、CapabilityManagerAgent）记 CapabilityManagerAgent 的 CHANGELOG。

### 新增（项目立项：软件测试 Agent TestEngineerAgent）

- **为什么建**：团队交给 agent 开发的软件项目反复出现回归（「改了这个功能、把别的功能改坏了」），开发 agent 自测只验证改动点、不知道改动影响面，且存在「代码改不绿就放宽断言」的自审风险。用户与 Kit 四轮讨论收敛（2026-09-06）：CI 与本地测试是纵深防御两层、测试定义权必须与实现权分离、裁决权必须交给远端 CI——这套体系需要一个专职角色持有用例定义权，用户拍板立项。拟人名 Hopper 查注册表无重复，名字取自 Grace Hopper（史上第一个计算机 bug 的发现者、「debug」一词推广者），隶属基础设施小组、独立于销售流水线。
- **改了什么**：按团队脚手架新建全套——根 `CLAUDE.md`（角色定义 + 三权分立工作原则：定义权归 Hopper / 实现权归开发 agent / 裁决权归 CI，含用例先行、先红后绿、分支开发 PR 门禁等纪律）、`README.md` / `README_cn.md` 双语（含三权分立设计、功能发布流程、服务项目清单）、`assets/logo.svg`（红 → 琥珀 → 绿三段渐变 #DC2626 → #D97706 → #059669，红绿灯意象 + 🐞，各项目配色查重无撞色）、`VERSION`（0.1.0）、`CHANGELOG.md`、`TODO.md` / `TODO-archive.md`（存量项目测试摸底一条待办）、`LICENSE.md`（MIT）、`.gitignore`、项目根 `AGENTS.md` 软链接指向 `CLAUDE.md`；`git init` 本地初始化（未建远程）。按 2026-09-03 修订的脚手架约定不建 `.claude/` 目录。关联同步：全局 `~/.claude/CLAUDE.md` 注册表加 Hopper 行 + 基础设施小组成员清单更新（CapabilityManagerAgent `claude/CLAUDE.md` 镜像随全局对齐）；xhqing 主页 roster 双语加行 + `scripts/update_traffic.py` 团队清单加 TestEngineerAgent + 预建徽章 JSON（细节记 xhqing 自己的 CHANGELOG）。
