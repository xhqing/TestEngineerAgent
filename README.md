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

> 🐞 **Hopper** — the software test engineer. Named after Grace Hopper, who found the first computer bug ever recorded (a moth in the Harvard Mark II, 1947) and popularized the word "debug". This agent hunts bugs for the whole team: it writes tests from GitHub Issues **before** development starts (red first, then green), and guards every project with a regression safety net.

[简体中文](README_cn.md)

TestEngineerAgent exists because of one recurring pain: agent-built projects kept breaking feature B while changing feature A (**regression**). The developing agent only verifies what it changed; nobody automatically checks that everything else still works. Hopper fixes this structurally — a test suite that runs in CI on every push, so regressions are caught by a red light **before** they merge into main.

---

## Separation of powers (the core design)

Testing only works if the tester is not the implementer. The design separates powers that usually live together:

| Power | Held by | Enforced by |
|---|---|---|
| **Definition** — writing tests | Hopper | The developing agent gets read-and-run-only access to all test files (hook-enforced: no creating, editing, deleting, or moving); if it thinks a test is wrong, it escalates for adjudication instead of editing it |
| **Implementation** — writing feature code | The developing agent | Hopper never edits feature code |
| **Merge adjudication** — declaring pass / fail | Remote CI (GitHub Actions) | Full test suite + type check; CI green triggers auto-merge into main with no human wait — a local green light is only a pre-check |
| **Release adjudication** — declaring ship / no-ship | The user | Real-world trial of prerelease builds (usage is acceptance); a formal release happens only once the user is satisfied |

Why so strict? Because an LLM that "can't turn the light green" is strongly tempted to widen the assertion instead of fixing the code. If the implementer can edit the tests, the safety net is made of paper.

---

## How a feature ships

1. **The requirement lands in an Issue.** The user opens a GitHub Issue whenever something breaks or is wanted, with reproduction steps / expected behavior — it doubles as the requirement doc, the acceptance criteria, and the seed of the tests.
2. **Tests come first on the feature branch (red).** Hopper reads the Issue and writes tests (one focused assertion for a small bug, a case group for a big feature) in the project's canonical test location, runs them to confirm red — proving the assertions actually bite — and commits to the branch with explicit user authorization.
3. **The developing agent implements on the same branch** — iterating against fast local test runs to green; test files are read-and-run-only for it (hook-enforced, even for adding tests).
4. **PR + CI gate:** the PR description carries `fixes #N`; remote CI runs the full suite (all old tests + the new ones) plus a type check, and green triggers auto-merge into main with the Issue auto-closed; main is never polluted.
5. **Prerelease trial (acceptance as a process):** the user installs an `rc.N` prerelease and uses it for real — checking each item against the Issue plus the feel, wording, and other dimensions that resist test cases; problems go through the fix loop (new Issue → Hopper writes a failing test first → fix merges → rc.N+1), and a formal release happens only once the user is satisfied.
6. **Every escape (a regression that slipped through) adds a test** — first prove the new test fails (red), then fix until green. The net only gets denser.

Main stays permanently green by design: main only ever receives complete red-green loops (tests and implementation land in the same PR), so a red main is always an incident, never "work in progress".

---

## What Hopper owns

- **New-feature acceptance tests** written from the Issue before development starts (red first, then green);
- **Regression net for existing projects** — built test by test from real incidents, prioritized by pain;
- **Legacy `test-cases/` migration** — moving cases from the retired directory layout into each project's canonical test location, verified via PR + CI;
- **Fix-loop test writing** — reproducing trial-found problems with failing tests before handing them to development;
- **Test infrastructure config** — commands that decide "what gets run" are off-limits to developing agents; changes go through Hopper or the user.

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
