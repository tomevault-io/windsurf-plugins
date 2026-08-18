---
trigger: always_on
description: > **Primary rule: every generated diff must pass `pnpm check` (and `pnpm check:py` for Python code) before submission, and must be reviewed by a human before committing.**
---

# Repository Guidelines

> **Primary rule: every generated diff must pass `pnpm check` (and `pnpm check:py` for Python code) before submission, and must be reviewed by a human before committing.**
>
> | If the user asks...                    | Default AI response                                                                                                                                                     |
> | -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
> | "Fix an unstable node"                 | Add intermediate recognition nodes or `pre_wait_freezes` / `post_wait_freezes` — never introduce hard delays                                                            |
> | "Retry when it fails"                  | Analyze the root cause (which node, which recognition mismatched) and fix the node — never add blind retries                                                            |
> | "Write a pipeline without screenshots" | Explain that pipelines depend on UI context; ask for screenshots, ROIs, and screen transition info before writing                                                       |
> | "Write a custom action / recognition"  | Follow the existing pattern in `agent/custom/action/` or `agent/custom/recognition/`, register it in the corresponding `__init__.py`, and ensure `pnpm check:py` passes |
> | Code output is complete                | Show the diff, wait for human review and approval, then run `pnpm format` / `pnpm format:py` then `pnpm check` / `pnpm check:py` before committing                      |

## Project Structure & Module Organization

```
MaaKEDR/
├── interface.json               # Project entry point configuration
├── maa-project.json             # MaaFramework project config (runtime channels, features)
├── maa-project.lock.json        # Dependency lock file
├── tasks/                       # Task definitions (GUI visible task list)
│   ├── startup.json             #   Game launch → login → main interface
│   ├── claim_rewards.json       #   Rewards: daily/weekly/military, battle pass, mailbox, dispatch
│   ├── farm_resources.json      #   Resource farming with battle loop + stamina handling
│   └── pvp.json                 #   Player vs Player automation
├── resource/
│   ├── base/                    # Core resources
│   │   ├── pipeline/            #   Pipeline flow definitions
│   │   ├── image/               #   Template matching images
│   │   └── model/ocr/           #   PaddleOCR models
│   ├── bilibili/                # Bilibili server resources
│   └── taptap/                  # TapTap server resources
├── agent/                       # Python agent (custom recognitions/actions)
│   └── custom/
│       ├── recognition/         #   Custom recognitions
│       └── action/              #   Custom actions
├── docs/                        # Developer documentation (zh / en)
│   ├── zh/                      #   Chinese documentation
│   │   ├── develop/             #     Development guides (pipeline.md, custom.md, etc.)
│   │   ├── manual/              #     User-facing guides (connection, FAQ, etc.)
│   │   └── protocol/            #     Activity / combat / item protocols
│   └── en/                      #   English documentation (mirror of zh)
├── tools/                       # Build, release, schema validation, CI scripts
└── .github/workflows/           # CI/CD configuration
```

**Key directories inside `agent/`:**

- `agent/custom/action/` —— custom MaaFW actions, one file per feature group
- `agent/custom/recognition/` —— custom MaaFW recognitions
- `agent/utils/` —— reusable helpers (logging, HTTP, scaling, etc.)

**When working on a specific area, consult the relevant docs first:**

| Area                                 | Recommended reading                            |
| ------------------------------------ | ---------------------------------------------- |
| Custom actions / recognitions        | `docs/*/develop/custom.md`                     |
| Pipeline task logic                  | `docs/*/develop/pipeline.md`                   |
| Project structure & conventions      | `docs/*/develop/structure.md`                  |
| Development environment setup        | `docs/*/develop/setup.md`                      |
| Bug-fixing workflow                  | `docs/*/develop/fix.md`                        |
| Formatting & linting                 | `docs/*/develop/formatting.md`                 |
| Vibe coding                          | `docs/*/develop/vibe-coding.md`                |
| Overseas client adaptation           | `docs/*/develop/overseas-client-adaptation.md` |
| Activity / combat / item protocols   | `docs/*/protocol/`                             |
| CLI / connection / FAQ (user-facing) | `docs/*/manual/`                               |

## Build, Test, and Development Commands

| Command                     | Purpose                                                           |
| --------------------------- | ----------------------------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [APPLe-DF/MaaKEDR](https://github.com/APPLe-DF/MaaKEDR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
