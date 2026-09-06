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

> 🐞 **Hopper** — the software test engineer. Named after Grace Hopper, who found the first computer bug ever recorded (a moth in the Harvard Mark II, 1947) and popularized the word "debug". This agent hunts bugs for the whole team: it turns requirements into executable acceptance tests **before** development starts, and guards every project with a regression safety net.

[简体中文](README_cn.md)

TestEngineerAgent exists because of one recurring pain: agent-built projects kept breaking feature B while changing feature A (**regression**). The developing agent only verifies what it changed; nobody automatically checks that everything else still works. Hopper fixes this structurally — a test suite that runs in CI on every push, so regressions are caught by a red light **before** they merge into main.

---

## Separation of three powers (the core design)

Testing only works if the tester is not the implementer. The design separates three powers that usually live together:

| Power | Held by | Enforced by |
|---|---|---|
| **Definition** — writing test cases | Hopper | The developing agent is forbidden to touch test cases; if it thinks a case is wrong, it escalates for adjudication instead of editing it |
| **Implementation** — writing feature code | The developing agent | Hopper never edits feature code |
| **Adjudication** — declaring pass / fail | CI (GitHub Actions) | A local green light is self-reported; only the remote CI verdict counts as third-party fact |

Why so strict? Because an LLM that "can't turn the light green" is strongly tempted to widen the assertion instead of fixing the code. If the implementer can edit the tests, the safety net is made of paper.

---

## How a feature ships

1. **Requirement → test cases (test-first).** Hopper interrogates the requirement until it is precise enough to assert (column order, date formats, empty-value behavior…), then writes acceptance cases covering happy paths, boundary values, and invalid inputs.
2. **Cases land on a feature branch, CI wired to run them.**
3. **The developing agent implements on the branch** — iterating against fast local test runs.
4. **PR gate:** CI runs the full suite (all old cases + the new ones). Green → merge. Red → keep working or discard the branch; main is never polluted.
5. **Every escape (a regression that slipped through) adds a case** — first prove the new case fails (red), then fix until green. The net only gets denser.

Main stays permanently green by design: a red main is always an incident, never "work in progress".

---

## What Hopper owns

- **New-feature acceptance cases** written before development starts;
- **Regression net for existing projects** — built case by case from real incidents, prioritized by pain;
- **Test directories and CI workflows** across the team's software repos;
- **Verdict reports** for delivery acceptance: remote CI results + list of newly added cases.

---

## Position in the team

Infrastructure squad (alongside Tinker, Prometheus, Hermes, Anvil, Atlas, Ada, Alfred), independent of the sales pipeline. Hopper serves every agent that ships software:

| Agent | Software projects |
|---|---|
| Atlas (FullStackEngineerAgent) | zcode-cli, zcode-vsce |
| Anvil (BackendEngineerAgent) | CC-BRIDGE |
| Alfred (DeviceStewardAgent) | ResourceMonitor |
| Hermes (NetOpsAgent) | XPilot |
| Markowitz (QuantStrategistAgent) | gridtrader |
| Ada (NeuralCoreAgent) | AgentCortex |

Boundary with Prometheus (CapabilityManagerAgent): test assets living in each project repo belong to Hopper; reusable testing methodology and templates are distributed through Prometheus's single open-source outlet.

---

## License & Attribution

Copyright (c) 2026 All Contributors. Licensed under the [MIT License](LICENSE.md).

**Attribution:** If you derive from or redistribute this project, please retain the copyright notice and license file, and credit the source: [TestEngineerAgent](https://github.com/xhqing/TestEngineerAgent).
