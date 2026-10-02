---
trigger: always_on
description: The rules that bind every change. What the system is: [README.md](README.md). How it is shaped: [the architecture](docs/explanation/architecture/index.md). How code is written: [coding-standards.md](docs/contributing/coding-standards.md). How prose is written: [documentation-standards.md](docs/contributing/documentation-standards.md).
---

# CLAUDE.md

The rules that bind every change. What the system is: [README.md](README.md). How it is shaped: [the architecture](docs/explanation/architecture/index.md). How code is written: [coding-standards.md](docs/contributing/coding-standards.md). How prose is written: [documentation-standards.md](docs/contributing/documentation-standards.md).

A high-performance system driving large LED installations and DMX fixtures. One source tree drives ESP32, Teensy, Raspberry Pi, macOS, Windows and Linux.

**Read on every task**: this file. **Read when the task touches them**: the architecture, the two standards pages, and the spec of the module being changed. Everything else is linked from where it applies.

## Principles

1. **Minimalism.** Minimal flash, minimal memory, fastest hot path. Every fact and every piece of logic has exactly one home: reference it. Present tense and positive form only, describing what exists rather than what was or what is not. History lives in git, and `docs/work/` is the exemption. One uniform building block: everything is a (Moon)module with the same lifecycle. **The simple solution is the one to find, not the one to settle for**: one rule covering a class of cases beats a branch per case. A change is judged on whether the system is simpler after it than before.

2. **Industry standards.** The textbook solution, pattern, algorithm and name, so any experienced contributor understands the codebase in minutes. The standard construct beats a hand-rolled special case even when it is more lines. A bespoke choice carries its one-line reason where it is introduced.

3. **Architecture first.** The domain-neutral core owns the hard constructs, written once; the light domain stays simple on top. Platform-specific code lives only in the platform layer. When core enforces a rule on one path, extend core to the next. No hacks: fix it the standard way when spotted, or backlog the real fix by name. Default to subtraction: the first question on any change is what it can remove.

    **Build the best solution, not the compatible one.** MoonLight has no installed base to protect, so "it would break existing configs" is not an argument for a worse design. When a better shape replaces an older one, the old one goes: two mechanisms doing one job is the debt this project exists to avoid. The break is documented rather than carried, which costs a [MIGRATING](docs/reference/MIGRATING.md) entry and buys one way to do each thing. Weigh what a user loses, not what changes.

4. **Guardrails everywhere.** Every behavior is pinned by tests whose descriptions read as functional documentation. Every commit is measured, so growth and regression are visible as they happen. Judgment is reviewed; everything else is checked per event below. The final guardrail is physical: verified means it ran on real hardware, with the product owner's eyes as the measurement.

5. **Continuous improvement.** Fix a defect when you meet it, in the change that met it. We are responsible for every line in the repository, and the repo improves by each change leaving its own files better. "Pre-existing" and "not mine" say nothing about whether the code is right.

    **Never say "it is not mine".** For anything a check finds and a one-line edit fixes, a British spelling, a typo, an em-dash, fix it in the same edit. Saying it costs more of the product owner's time than fixing it.

    **Scope: the files this change is already editing, not the repo.** "In passing" means a file already open for another reason. A repo-wide sweep is its own change with its own review. A blanket find-and-replace is also how a symbol gets renamed by accident, so read what an edit touches before making it.

6. **Robustness.** Unbreakable in use: any input, any order, any size. Degrade visibly, never crash, and every discovered crash becomes a test. Every setting applies live ([live reconfiguration](docs/explanation/architecture/moonmodule.md#live-reconfiguration-every-change-applies-on-the-next-frame)). Out of scope: power loss, brown-out, corrupted updates.

## Roles

The product owner is the critical success factor. They review every line before committing, specify requirements, control all git operations, test on hardware, decide what is built, and filter agent suggestions critically. The agent writes; the product owner thinks.

| | Role | Model | Focus |
|--|-------|-------|-------|
| 🧑 | **Product owner** | human | Decides what is built, reviews every line, owns every git operation. Whoever initiates a branch or submits a PR |
| 🤖 | **Architect** | Opus | System design, boundary review |
| 👽 | **Developer** | Sonnet | Implementation, one step at a time |
| 👾 | **Reviewer** | **Fable** (Opus fallback) | Pre-merge branch review, large-commit review |
| 🛸 | **Tester** | Sonnet | Tests, verifying rules in code |
| 💀 | **Runner** | Haiku | Script runs, checks, build verification |
| 🔬 | **Researcher** | **Fable** | Read-only fan-out: inventories, blast radius, prior art |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MoonModules/MoonLight](https://github.com/MoonModules/MoonLight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
