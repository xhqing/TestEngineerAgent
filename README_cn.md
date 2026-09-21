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

> 🐞 **Hopper** —— 软件测试工程师。名字取自 Grace Hopper：史上第一个计算机 bug（1947 年哈佛 Mark II 继电器里的那只蛾子）的发现者、「debug」一词的推广者。这个 agent 替全团队找 bug：开发开始**之前**基于 GitHub Issue 把测试写出来（先红后绿），并用回归防护网守住每个项目。

[English](README.md)

TestEngineerAgent 因一个反复出现的痛点而生：交给 agent 开发的软件项目，经常「改了这个功能、把别的功能改坏了」（**回归**）。开发 agent 改完只验证改动点，没有谁在自动确认「其它功能还正常」。Hopper 从结构上解决这件事——一套随每次 push 在 CI 里全量运行的测试，让回归在**合并进 main 之前**就被红灯抓住。

---

## 权力分立（核心设计）

测试要成立，测试者就不能是实现者。本设计把通常捏在一起的几份权力拆开：

| 权力 | 归属 | 保障 |
|---|---|---|
| **定义权**——写测试 | Hopper | 开发 agent 对测试文件全量只读 + 可运行（hook 工具强制禁止增删改）；认为测试有误只能上报裁决，不许自己动手改 |
| **实现权**——写功能代码 | 开发 agent | Hopper 不写、不改功能代码 |
| **合并裁决权**——判定过没过 | 远端 CI（GitHub Actions） | 全量测试 + 类型检查，CI 绿即 auto-merge 合并进 main，不等人工；本地绿灯只是预检 |
| **发布裁决权**——判定发不发 | 用户 | 预发布版本实际试用（体验即验收），满意才正式发版 |

为什么要这么严？因为「绿灯点不亮」的 LLM 有很强的动机去放宽断言而不是修代码。实现者一旦能改测试，防护网就是纸糊的。

---

## 一个功能是怎么发布的

1. **需求落进 Issue**。用户遇到问题 / 想要功能随手开 Issue，正文写清复现步骤 / 预期行为——它同时是需求文档、验收标准和测试的雏形。
2. **功能分支上测试先行（先红）**。Hopper 读 Issue 出测试（小 bug 一条聚焦断言，大功能成组用例），写在项目正式测试位置，自跑确认红——验证断言真的在测东西，经用户授权 commit 进分支。
3. **开发 agent 在同一分支实现**——对着本地快速测试迭代到全绿；测试文件对它全量只读 + 可运行（hook 强制，补测试也不行）。
4. **PR + CI 门禁**：PR 描述带 `fixes #N`，远端 CI 跑全量测试（所有旧测试 + 本次新测试）+ 类型检查，绿即 auto-merge 合并进 main、Issue 自动关闭；main 始终不被污染。
5. **预发布试用（验收是过程）**：用户安装 `rc.N` 预发布版本实际使用——按 Issue 逐条核对 + 体验手感、文案这些难以用例化的维度；发现问题走修复循环（新 Issue → Hopper 先写失败测试 → 修复合并 → rc.N+1 再试用），满意才正式发版。
6. **每漏网一次（回归事故）就补一条测试**——先证明新测试能红（真能抓住 bug），再修到绿。网只增不減。

main 永远绿是设计出来的：main 只见「红绿闭环过」的完整状态（测试与实现同 PR 进 main），main 一红必是事故，绝不是「开发中」。

---

## Hopper 管什么

- **新功能的验收测试**：开发开始前基于 Issue 写好（先红后绿）；
- **存量项目的回归网**：从事故驱动一条条补，按痛的程度排优先级；
- **存量 `test-cases/` 目录迁移**：旧目录体系的用例搬进项目正式测试位置，迁移经 PR + CI 验证；
- **修复循环出题**：试用发现的问题先写失败测试复现，再交开发修；
- **测试体系配置**：测试命令等「跑什么测试」的配置不归开发动，要改走 Hopper 或用户。

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
