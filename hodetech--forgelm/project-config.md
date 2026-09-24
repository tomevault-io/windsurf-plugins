---
trigger: always_on
description: > **Audience:** Codex (and other AI coding agents) working on ForgeLM. Complements — does not replace — the human-facing [CONTRIBUTING.md](CONTRIBUTING.md) and [docs/standards/](docs/standards/).
---

# AGENTS.md — Project Guidance for AI Agents

> **Audience:** Codex (and other AI coding agents) working on ForgeLM. Complements — does not replace — the human-facing [CONTRIBUTING.md](CONTRIBUTING.md) and [docs/standards/](docs/standards/).

## What ForgeLM is (in one line)

A **config-driven, enterprise-grade LLM fine-tuning toolkit** — YAML in, fine-tuned model + compliance artifacts out. Drives the same workflow from a terminal, a notebook, or a CI/CD pipeline step. Covers SFT → DPO → SimPO → KTO → ORPO → GRPO, with integrated safety evaluation, EU AI Act compliance, and auto-revert on quality regression.

Not a framework for training from scratch. Not an inference engine. Not a GUI. Read [docs/product_strategy.md](docs/product_strategy.md) for the 5-minute background.

## What you must read before editing code

**Every time, in this order:**

1. **[docs/standards/README.md](docs/standards/README.md)** — index of all engineering standards
2. **The specific standard** matching what you're about to change:
   - Python code → [coding.md](docs/standards/coding.md) + [architecture.md](docs/standards/architecture.md)
   - **Any `re.compile` / regex change → [regex.md](docs/standards/regex.md)** (ReDoS exposure, fixture fragmentation, the 8 hard rules distilled from Phase 11/11.5/12 review cycles)
   - Error paths → [error-handling.md](docs/standards/error-handling.md)
   - Anything with output → [logging-observability.md](docs/standards/logging-observability.md)
   - Tests → [testing.md](docs/standards/testing.md)
   - Docs → [documentation.md](docs/standards/documentation.md) + [localization.md](docs/standards/localization.md)
   - PR / review → [code-review.md](docs/standards/code-review.md)
   - Release → [release.md](docs/standards/release.md)
3. **[CONTRIBUTING.md](CONTRIBUTING.md)** — the human-facing summary
4. **The relevant roadmap file** — if implementing a planned phase, find it under [docs/roadmap/](docs/roadmap/)

Do not invent conventions. If you cannot find the pattern for what you're about to add, ask the user — don't guess.

## Skills

When a task maps to a common pattern, invoke the matching skill from [.agents/skills/](.agents/skills/):

| Task | Skill |
|---|---|
| Adding a YAML config field | [add-config-field](.agents/skills/add-config-field/SKILL.md) |
| Adding a larger trainer / evaluator / module feature | [add-trainer-feature](.agents/skills/add-trainer-feature/SKILL.md) |
| Writing tests | [add-test](.agents/skills/add-test/SKILL.md) |
| Updating bilingual docs (EN ↔ TR) | [sync-bilingual-docs](.agents/skills/sync-bilingual-docs/SKILL.md) |
| Reviewing a PR (own or peer) | [review-pr](.agents/skills/review-pr/SKILL.md) |
| Cutting a release | [cut-release](.agents/skills/cut-release/SKILL.md) |

Each skill's `SKILL.md` has the full checklist. Follow it; don't skip steps to save time.

## Repository structure at a glance

```text
ForgeLM/
├── forgelm/                 # Source code: ~21 single-file modules + 4 sub-packages
│   ├── cli/                 # CLI package (Phase 15 split): _parser, _dispatch,
│   │                        # _exit_codes, subcommands/{ingest, audit, chat,
│   │                        # export, deploy, quickstart, doctor, cache,
│   │                        # purge, reverse_pii, approve, approvals,
│   │                        # safety_eval, verify_audit, verify_annex_iv,
│   │                        # verify_gguf, ...}
│   ├── data_audit/          # Audit package (Phase 14 split): _orchestrator,
│   │                        # _aggregator, _streaming, _simhash, _minhash,
│   │                        # _pii_regex, _pii_ml, _secrets, _quality,
│   │                        # _croissant, _summary, _splits
│   ├── wizard/              # Interactive --wizard config generation: _collectors,
│   │                        # _orchestrator, _state, _byod, _io, _defaults.json
│   ├── config.py            # Pydantic schemas (23 models)
│   ├── trainer.py           # TRL wrapper (SFT/DPO/SimPO/KTO/ORPO/GRPO)
│   ├── model.py             # HF + PEFT model loading
│   ├── data.py              # Dataset loading + format detection
│   ├── ingestion.py         # Raw docs → SFT JSONL (`forgelm ingest`)
│   ├── safety/              # Safety package (post-v0.9.1 split): _types,
│   │                        # _inputs, _generate, _classifier,
│   │                        # _score_classification, _score_generation,
│   │                        # _gates, _results, _orchestrator
│   ├── compliance.py        # EU AI Act Articles 9-17 + Annex IV + GDPR purge / reverse-pii primitives
│   ├── webhook.py           # Slack/Teams notifications (5-event vocabulary)
│   ├── grpo_rewards.py      # Built-in GRPO format/length shaping reward fallback
│   ├── _http.py             # SSRF-guarded HTTP chokepoint (safe_post / safe_get)
│   ├── _version.py          # `__version__` + `__api_version__` (decoupled)
│   └── ...                  # benchmark, judge, merging, synthetic,
│                            # quickstart, model_card, fit_check, deploy, chat,
│                            # export, inference, results, utils

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HodeTech/ForgeLM](https://github.com/HodeTech/ForgeLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
