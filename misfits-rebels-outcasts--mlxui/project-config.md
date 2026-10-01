---
trigger: always_on
description: Guidance for working in this repository.
---

# CLAUDE.md

Guidance for working in this repository.

## What this is

**MLXUI** is a native macOS SwiftUI app for Apple Silicon that does two things:

1. **Browse, compare, install and run local MLX models** — a bundled catalog sourced from
   HuggingFace, filtered/sorted by hardware fit (RAM, chip bandwidth), installed by downloading
   files directly from HF, and run locally via MLX (LLM chat, ASR, TTS, VLM, OCR, embeddings,
   diffusion, segmentation, video, upscale).
2. **Run those models as workflows** — **CAT Flow**, reached at *Automate → AI Workflows*. A
   `.cat` file is a numbered list of `Asset → Transform → Asset` rows that a runtime executes top
   to bottom. `MLXUI/FlowKit/` is a Swift port of the `catflow-mlx` Python runtime.

The second half arrived through the CAT Flow merge (`RSI/DelegateMergeBacklog.md`, phases R1…R16)
and is now the larger surface. A change to model support is usually *also* a change to what flows
can run — see "CAT Flow" below.

- Platform: **macOS only**, deployment target **14.0**, set in `Config/Shared.xcconfig`.
- **Two editions** (`Design/dual-distribution.md`):

  | Target | Product | Distribution | Sandbox | Flag |
  |---|---|---|---|---|
  | `MLXUI` | "AI Browser" | Mac App Store | **yes** — app-sandbox, network.client, user-selected read-write | `APPSTORE_BUILD` |
  | `MLXUI-Direct` | "AI Browser Pro" | Developer ID + notarized | **no** — Hardened Runtime only | `DIRECT_BUILD` |

  The Direct edition compiles in the shell / filesystem / subprocess agent tools; the App Store
  target never has those files in its membership, so the sandboxed binary cannot contain them.
  `FlowKit/CapabilityGate.swift` refuses capability classes by channel at run time.
- Build settings live in `Config/{Shared,AppStore,Direct}.xcconfig`, wired as each target's
  `baseConfigurationReference`. **A target-level setting in Xcode's editor overrides the
  xcconfig** — check there first when a setting doesn't take.
- **No analytics. The app itself sends nothing, ever** — no telemetry, no phone-home, no
  developer-operated server. It has none.
- **What can leave the machine, and only when a flow or the user asks for it** (revised
  2026-09-12, after the off-machine stream — the old one-line "no data collection" claim is no
  longer accurate on its own):

  | Leaves | Carrying | Under whose account | Since |
  |---|---|---|---|
  | HuggingFace | the model id being fetched; the user's HF token if set | the user's | v1 |
  | `Web Fetch` · `HTTP Get` · `Fetch Feed` · `Download File` | the URL the row names | none | CFM-R12-9 |
  | `Web Search` → Tavily or Brave | the query text | **the user's own API key** | WS |
  | a remote model row → Anthropic / OpenAI / DeepSeek / Together / Groq / OpenRouter / Cohere | **the row's prompt — i.e. the user's document content** | **the user's own API key** | RM |
  | a LAN endpoint (`egress: "lan"`) | the row's prompt | none; the user's own machine | RM |

  Apple Foundation Models runs **on-device** — nothing leaves. Keys live in the macOS Keychain,
  are never written into a `.cat` file, and are never logged. A flow binding any provider must
  declare `offdevice` on line one or fail check with **E120**, whichever the egress.
  **Before any App Store submission with remote enabled, read
  `RSI/RM-6-privacy-disclosure.md`** — the nutrition-label decision lives there.

### Mission

The app supports a **growing list of MLX models**, and a model isn't "done" until it can be
installed and actually run. That now means two things, both required:

- **A Run UI** matched to the modality (chat, ASR, TTS, image Q&A, OCR, embeddings, diffusion, …),
  built from the module template in `Design/aisdk-aiui-architecture.md`.
- **Reachability from AI Workflows** — a `CatalogBridge` row, a curated manifest under
  `Resources/CatFlow/models/`, and a task the executor genuinely serves. This is step **AM-W** of
  `RSI/prompts/add-model.md` (the `/add-mlxui` intake), and it is the step most often forgotten:
  a model can install and run from Browse while every `.cat` row naming it refuses.

When an architecture has no MLX runner, it's flagged in `Core/ModelSupport.swift` and queued for an
iterative port rather than left silently broken.

## The RSI process — read this before starting work

Work is driven from `RSI/`, one reviewable cycle at a time. Not ad hoc.

| File | Role |
|---|---|
| `RSI/policies.md` | **Read first, every cycle.** Current autonomy level (**L2** — one task per cycle, scorecard-first review; promoted from L1 on 2026-09-04, journal `2026-172`) and the standing guardrails. |
| `RSI/backlog.md` | The owner's backlog. `/add-mlxui` appends `AM-*` intake groups here. |
| `RSI/DelegateMergeBacklog.md` | The CAT Flow merge phases (R1…R16), delegated for external implementation. |
| `RSI/journal/` | One entry per task — what was done, what the gates said, what was found. |
| `RSI/evals/smoke-checklist.md` | Human-run smoke rows. A model or flow isn't verified until one passes. |
| `RSI/prompts/add-model.md` | The add-a-model intake (`/add-mlxui`), Steps A–D. |
| `Design/RSI.md` | The autonomy ladder (§3) and guardrails (§9) that `policies.md` instantiates. |

The guardrails that bite most often:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Misfits-Rebels-Outcasts/MLXUI](https://github.com/Misfits-Rebels-Outcasts/MLXUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
