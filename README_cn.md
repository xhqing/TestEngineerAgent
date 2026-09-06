<div align="center">
  <img src="assets/logo.svg" alt="TestEngineerAgent" width="640">
</div>

<div align="center">

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE.md)
[![Version](https://img.shields.io/badge/Version-0.1.0-blue.svg)](VERSION)
[![Type](https://img.shields.io/badge/Type-AI%20Agent-FF1493.svg)](#)
<img src="https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/xhqing/xhqing/main/traffic/badges/TestEngineerAgent.json" alt="Visits/day (14d)" />

</div>

# TestEngineerAgent

> 🐞 **Hopper** —— 软件测试工程师。名字取自 Grace Hopper：史上第一个计算机 bug（1947 年哈佛 Mark II 继电器里的那只蛾子）的发现者、「debug」一词的推广者。这个 agent 替全团队找 bug：在开发开始**之前**把需求变成可执行的验收测试，并用回归防护网守住每个项目。

[English](README.md)

TestEngineerAgent 因一个反复出现的痛点而生：交给 agent 开发的软件项目，经常「改了这个功能、把别的功能改坏了」（**回归**）。开发 agent 改完只验证改动点，没有谁在自动确认「其它功能还正常」。Hopper 从结构上解决这件事——一套随每次 push 在 CI 里全量运行的测试，让回归在**合并进 main 之前**就被红灯抓住。

---

## 三权分立（核心设计）

测试要成立，测试者就不能是实现者。本设计把通常捏在一起的三份权力拆开：

| 权力 | 归属 | 保障 |
|---|---|---|
| **定义权**——写测试用例 | Hopper | 开发 agent 禁止改动测试用例；认为用例有误只能上报裁决，不许自己动手改 |
| **实现权**——写功能代码 | 开发 agent | Hopper 不写、不改功能代码 |
| **裁决权**——判定过没过 | CI（GitHub Actions） | 本地绿灯是自报，只有远端 CI 的判定才算第三方事实 |

为什么要这么严？因为「绿灯点不亮」的 LLM 有很强的动机去放宽断言而不是修代码。实现者一旦能改测试，防护网就是纸糊的。

---

## 一个功能是怎么发布的

1. **需求 → 测试用例（用例先行）**。Hopper 把需求盘问到足够精确（列顺序、日期格式、空值行为……），再写验收用例，覆盖正常路径、边界值、异常输入。
2. **用例落进 feature 分支，CI 接管执行。**
3. **开发 agent 在分支上实现**——对着本地快速测试迭代。
4. **PR 门禁**：CI 跑全量用例（所有旧用例 + 本次新用例）。绿 → 合并；红 → 继续改或丢弃分支，main 始终不被污染。
5. **每漏网一次（回归事故）就补一条用例**——先证明新用例能红（真能抓住 bug），再修到绿。网只增不減。

main 永远绿是设计出来的：main 一红必是事故，绝不是「开发中」。

---

## Hopper 管什么

- **新功能的验收用例**：开发开始前写好；
- **存量项目的回归网**：从事故驱动一条条补，按痛的程度排优先级；
- **各项目的测试目录与 CI workflow**；
- **交付验收报告**：远端 CI 结果 + 新增用例清单。

---

## 在团队中的位置

基础设施小组成员（与 Tinker / Prometheus / Hermes / Anvil / Atlas / Ada / Alfred 同组），独立于销售流水线。Hopper 服务所有产出软件的 agent：

| Agent | 软件项目 |
|---|---|
| Atlas（FullStackEngineerAgent） | zcode-cli、zcode-vsce |
| Anvil（BackendEngineerAgent） | CC-BRIDGE |
| Alfred（DeviceStewardAgent） | ResourceMonitor |
| Hermes（NetOpsAgent） | XPilot |
| Markowitz（QuantStrategistAgent） | gridtrader |
| Ada（NeuralCoreAgent） | AgentCortex |

与 Prometheus（CapabilityManagerAgent）的边界：各项目仓库里的测试资产归 Hopper；可复用的测试方法论与模板，经 Prometheus 的通用能力单一出口分发。

---

## 许可与署名

版权所有 (c) 2026 All Contributors，基于 [MIT License](LICENSE.md) 授权。

**署名要求**：如你基于本项目衍生或再分发，请保留版权声明与许可文件，并注明来源：[TestEngineerAgent](https://github.com/xhqing/TestEngineerAgent)。
