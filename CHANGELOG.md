# CHANGELOG

本文件记录本项目（TestEngineerAgent / Hopper）每次文件增删改查的变更，写清「为什么改」和「改了什么」。版本号以项目根 `VERSION` 文件为唯一权威（当前 0.1.0）。

## [0.1.0] - 2026-09-06

### 新增（项目立项：软件测试 Agent TestEngineerAgent）

- **为什么建**：团队交给 agent 开发的软件项目反复出现回归（「改了这个功能、把别的功能改坏了」），开发 agent 自测只验证改动点、不知道改动影响面，且存在「代码改不绿就放宽断言」的自审风险。用户与 Kit 四轮讨论收敛（2026-09-06）：CI 与本地测试是纵深防御两层、测试定义权必须与实现权分离、裁决权必须交给远端 CI——这套体系需要一个专职角色持有用例定义权，用户拍板立项。拟人名 Hopper 查注册表无重复，名字取自 Grace Hopper（史上第一个计算机 bug 的发现者、「debug」一词推广者），隶属基础设施小组、独立于销售流水线。
- **改了什么**：按团队脚手架新建全套——根 `CLAUDE.md`（角色定义 + 三权分立工作原则：定义权归 Hopper / 实现权归开发 agent / 裁决权归 CI，含用例先行、先红后绿、分支开发 PR 门禁等纪律）、`README.md` / `README_cn.md` 双语（含三权分立设计、功能发布流程、服务项目清单）、`assets/logo.svg`（红 → 琥珀 → 绿三段渐变 #DC2626 → #D97706 → #059669，红绿灯意象 + 🐞，各项目配色查重无撞色）、`VERSION`（0.1.0）、`CHANGELOG.md`、`TODO.md` / `TODO-archive.md`（存量项目测试摸底一条待办）、`LICENSE.md`（MIT）、`.gitignore`、项目根 `AGENTS.md` 软链接指向 `CLAUDE.md`；`git init` 本地初始化（未建远程）。按 2026-09-03 修订的脚手架约定不建 `.claude/` 目录。关联同步：全局 `~/.claude/CLAUDE.md` 注册表加 Hopper 行 + 基础设施小组成员清单更新（CapabilityManagerAgent `claude/CLAUDE.md` 镜像随全局对齐）；xhqing 主页 roster 双语加行 + `scripts/update_traffic.py` 团队清单加 TestEngineerAgent + 预建徽章 JSON（细节记 xhqing 自己的 CHANGELOG）。
