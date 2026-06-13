---
trigger: always_on
description: Shinken is an **AI-native, cross-platform sandbox runtime + control plane + control panel for
---

# CLAUDE.md — guide for AI coding sessions in this repo

Shinken is an **AI-native, cross-platform sandbox runtime + control plane + control panel for
computer-use agents** — the runtime that benchmarks and harnesses plug into. See [README.md](README.md) and
[`docs/`](docs/README.md). The design corpus is complete; the **implementation is a proven
Linux/X11 vertical slice** — handshake/auth, pointer+keyboard actions, pixel observation
(screenshot + real-time screencast + bandwidth levers + focused-window capture),
Docker disk-tier checkpoint/fork/resume + the privileged-only **CRIU memory tier**
(live process+memory forks) + `run_eval_forked`, a local capability-gateway shim,
the **guest structured-observation engine v1 (Linux/AT-SPI: `observe` with stable element ids,
tree-text diff, settle; guest-side `element_ref` actions + `invoke_action`/`set_value`)**,
and a Python SDK, all under live CI, plus a **macOS engine v1** (native CoreGraphics/CGEvent
backend in `shinkend`; capture+input built, local-only proof — no mac CI, AX designed-only;
[docs/engineering/macos-engine.md](docs/engineering/macos-engine.md)). **Production permission
enforcement, `.skn` recording/playback, the control plane, and the rest of cross-platform
(Windows, Wayland, macOS AX) are designed-only and not yet built**. The a11y-coverage spike (#2)
has been **measured (E5) — verdict: hybrid per-window structured + pixel fallback, so D3's
structured-default stays Provisional** (canvas is a measured zero with a change-blind diff,
Electron is measured on both CDP and forced AT-SPI; games/native-GL still unmeasured; the
in-guest CDP backend and UIA/AX tiers remain unbuilt).
**[`docs/engineering/status.md`](docs/engineering/status.md) is the authoritative built-vs-designed map — read it before
trusting present-tense claims in the vision docs. This file's status summary must track
status.md; reconcile both when either changes.**

## ⛔ The one hard rule: this is a PUBLIC open-source project

This is a **public, vendor-neutral OSS project** (regardless of where it sits in a local
checkout). Anything committed is world-readable.

- **NEVER** put confidential or company-internal material in tracked files (`docs/`, `notes/`,
  `README`, code). No internal platform names, no internal links, nothing marked confidential.
- Internal/private design references must stay out of tracked files. Do not link to private working
  areas or use them as public documentation sources.
- **Public** vendor product facts (e.g. NVENC, NICE DCV, vGPU/MIG, GPU-TEE) ARE fine in docs when
  cited from public sources — the project stays vendor-neutral and runs on any cloud.
- Do not run internal-only tooling (e.g. company intranet search) for this project.

## Layout

| Path | Tracked? | What |
|------|----------|------|
| `docs/` | ✅ | Authoritative docs: vision, PRD, architecture, OSWorld teardown, landscape, ADRs (D1–D15), roadmap, glossary, isolation & capability note, economics, Phase-0 plan, ACI spec, operation layer |
| `notes/` | ✅ | 9 working notes: per-domain deep dives, open questions, sources |
| `README.md`, `LICENSE` (Apache-2.0) | ✅ | front matter |
| `schema/` | ✅ | ACI JSON Schema (`aci.schema.json`) |
| `shinkend/` | ✅ | Rust Guest Runtime (`shinkend`) |
| `sdk/python/` | ✅ | Python SDK and CLI (incl. `shinken/integrations/` — swerex/uni-agent, CUA-Gym, Agentix, ProRL-Agent-Server, NeMo-Gym interop adapters; `shinken/backends/` operation-layer backends — drive the ACI over third-party computer-control systems, e.g. trycua/cua) |
| `images/linux/` | ✅ | Local Linux Sandbox image |
| `examples/` | ✅ | Runnable interop examples (`gym_rollout.py`, `cua_gym_shinken.py`, `agentix_shinken.py`, `uniagent_shinken.py` — scripted, no model API) |
| `benchmarks/` | ✅ | Rerunnable local benchmark suites (incl. `bench_client_scale.py` N=3096 client plane) + tracked raw results (`results/*.json`, one-off WAN CSVs in `results/remote/`); ALL figures land in `docs/assets/bench/` (regenerate: `replot.py` + `plot_remote.py`); methodology in `docs/engineering/benchmarks.md`, headline report in `docs/benchmarks/README.md`; plus the agent-quality STUDY harness (`bench_agent_quality.py` — codec tier × task success over fork-identical episodes, `docs/engineering/agent-quality-study.md`; not in `run_all.sh`, needs a model endpoint) |
| `references/` | 🚫 git-ignored | 13 cloned prior-art repos studied for design (OSWorld, cua, codex, anthropic-quickstarts, neko, OpenAdapt, e2b-desktop, UI-TARS-desktop, OmniParser; + 2026-06: uni-agent, CUA-Gym, Agentix, ProRL-Agent-Server); provenance + re-clone in `references/README.md` (tracked) |

## Conventions

- **The public design canon is `docs/design/tech-decisions.md`** — decisions are numbered **D1–D15**.
  When changing a design decision, update the relevant ADR and reconcile sibling docs to the same
  D-number.
- Naming (use consistently): **Shinken** (platform), **Sandbox** / **Session**, **Guest Runtime**
  (`shinkend`), **ACI** (Agent-Computer Interface), **Operator**, **Control Plane** / **Control
  Panel**, **Substrate/Provider**, **Capability** / **Capability Manager**, **`.skn`** (replay
  bundle), the control/event/media **planes**. See [docs/design/glossary.md](docs/design/glossary.md).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Meirtz/Shinken](https://github.com/Meirtz/Shinken) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
