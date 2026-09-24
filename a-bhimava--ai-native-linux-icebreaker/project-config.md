---
trigger: always_on
description: > This file is read automatically by Claude Code at session start.
---

# CLAUDE.md — AI-Native OS Project

> This file is read automatically by Claude Code at session start.  
> It encodes the architectural invariants, security constraints, and anti-patterns for this codebase.  
> **Read this before writing a single line of code or proposing any change.**

---

## What This Project Is

An AI-Native Ubuntu fork in which a locally-running LLM is a first-class OS citizen. Users express intent in natural language; the AI translates it into safe, auditable system operations executed against the Linux kernel.

**Core documents:**
- [`AI_Native_OS_Whitepaper.md`](./AI_Native_OS_Whitepaper.md) — architecture, design decisions, KPIs (the source of truth)
- [`docs/IMPLEMENTATION_PLAN.md`](./docs/IMPLEMENTATION_PLAN.md) — build order, phase gates, failure mode register, risk mitigations

Read both before making any architectural change.

---

## Directory Map

```
/
├── AI_Native_OS_Whitepaper.md     # Definitive architecture reference
├── CLAUDE.md                      # This file
├── docs/
│   └── IMPLEMENTATION_PLAN.md     # Detailed build plan + failure register
├── privileged-brain/              # Fine-tuning pipeline for the Privileged Brain
│   ├── scripts/                   # Python training scripts (sft_train.py, dpo_train.py, etc.)
│   ├── inference/grammar/         # GBNF grammar for constrained decoding
│   ├── data/                      # Training data (synthetic + processed)
│   └── *.sh                       # Shell wrappers for the training pipeline
├── src/
│   └── mcpd/                      # Rust MCP daemon (Phase 1 — COMPLETE, on main)
├── dual-brain/                    # Phase 2 deployable bundle (Python)
│   ├── controller/                #   - Controller package (backends, audit, hitl, session, repl, risk, trust)
│   ├── gui_agent/                 #   - GUI Agent: AT-SPI client, screenshot manager, app APIs, Landlock sandbox
│   ├── rpa_bridge/                #   - RPA Bridge: Robot Framework, workflow gen, image match, Landlock+uinput sandbox
│   ├── terminal/                  #   - Textual TUI: split-pane, companion panel, input router, CoT rendering
│   ├── scripts/                   #   - Operator scripts (export_mcpd_catalogue.py, start_pb.sh, start_qb_local.sh)
│   ├── docs/phase2/               #   - Mirror of project Phase 2 planning docs
│   └── README.md                  #   - "tar dual-brain → scp → extract on VM" deploy workflow
├── shell/                         # V1 shell trigger — preserved untouched; coexists with Controller
├── backups/                       # Local snapshots of VM state (e.g. backups/jun4/)
├── cx-distro/                     # ISO build pipeline (6-stage build.sh, Dockerfile, distro config — PRs #15–#21)
│   └── rebuild/                   #   - VM deploy + ISO rebuild scripts (deploy.sh, rebuild.sh, monitor.sh)
├── dual-brain/gui/                # GTK4/LibAdwaita GUI apps (chatbot, settings, audit viewer)
│   ├── chatbot/                   #   - Copilot-style NL chat window (window.py, input_bar.py, message_list.py)
│   ├── settings/                  #   - Settings window (backend config, model selection)
│   ├── tray/                      #   - System tray indicator
│   └── daemon_client.py           #   - GtkDaemonClient: wraps DaemonClient with GLib.idle_add for GTK thread safety
└── models/
    └── checksums.sha256           # SHA-256 of all GGUF model weight files
```

## Phase Status

| Phase | Status | Module |
|---|---|---|
| Phase 0 | Complete | Env setup, models, MCP handshake |
| Phase 1 | Complete | mcpd Rust daemon (M1.0–M1.10, all exit gates green on Linux) |
| Phase 2 | Complete | Dual-Brain Controller (M2.0–M2.14; gates G1–G11 green; merged to `main`) |
| Phase 3 | Complete (folded into Phase 1) | Landlock + Seccomp-BPF + COW landed alongside the mcpd tools (M1.3 / M1.4 / M1.5); exit criteria met within Phase 1; kept as a heading for whitepaper continuity |
| Phase 4 | Complete | PB finalized: `run7_cot_q4km.gguf` — 100% adversarial refusal, 95.5% grammar-valid MCP, 940 MB, checksummed. run8 continued-tune rejected (safety regression). Pipeline in `privileged-brain/` |
| Phase 5 | **Complete** | UX + Graduated Determinism. P0: hardened HITL, keymap, trust store, tier-2 review, audit hash-chain (PR #9). P1-A: env scrub, cost/limits, TOCTOU (PR #10). P1-BCD: audit viewer TUI, streaming, undo scaffold (PR #11). P2: OpenAI backend + verifier voting (PR #12), presenter registry + screen-reader + GTK (PR #13), daemon/client split + systemd (PR #14). 1347 tests, G1–G11 + G5.1–G5.P2b green. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [a-bhimava/AI-Native-Linux-Icebreaker](https://github.com/a-bhimava/AI-Native-Linux-Icebreaker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
