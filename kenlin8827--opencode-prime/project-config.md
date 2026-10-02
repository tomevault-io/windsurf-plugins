---
trigger: always_on
description: This repo is **OpenCode Prime (OCP)** — a multi-agent configuration suite for [OpenCode](https://opencode.ai). It ships agent prompts, plugins, profiles, and an installer into `~/.config/opencode`. See `DEVELOPING.md` for architecture and contribution details.
---

# OpenCode Agent & Development Guidelines (AGENTS.md)

This repo is **OpenCode Prime (OCP)** — a multi-agent configuration suite for [OpenCode](https://opencode.ai). It ships agent prompts, plugins, profiles, and an installer into `~/.config/opencode`. See `DEVELOPING.md` for architecture and contribution details.

---

## 0. Design Principle — Architectural Legitimacy

This is an open-source project: **every design decision must withstand public scrutiny.** Architectural legitimacy outranks implementation convenience.

- **Capability-preserving efficiency** — reduce token cost by removing redundancy, not by weakening outcomes. A feature that impairs correct delivery, verification, or informed judgment is a regression; prefer scoped, on-demand capability over permanent context exposure.
- **Native mechanisms first** — prefer the platform's designed extension points (skills for on-demand disclosure, command files for slash commands, hooks for runtime behavior) over ad-hoc workarounds. A hack that "works" but defies the platform's design is a liability that invites criticism.
- **Refactor over patch** — when a mechanism is structurally wrong (e.g., static prompt injection where on-demand loading belongs), fix the architecture. Do NOT accumulate compensating hacks on top of a flawed foundation.
- **Defendability gate** — refactoring cost never justifies shipping a design the maintainers themselves cannot defend in public. If it would be embarrassing to explain, redesign it before merging.
- **Top-tier engineering floor** — code that fails top-tier engineering **quality** (correctness, performance, security, testability, type safety, error/edge-case handling) **or philosophy** (maintainability, defensibility, platform-native design, simplicity, fit with project design principles) MUST be triaged on encounter (refactor inline / file-as-issue / explicit-out-of-scope, by impact on current task) and MUST clear both (a) the explicit rule set (`cp-<slug>` baseline + per-language hard rules) and (b) the Defendability gate above. "Do less / lazy / pragmatic / good-enough" rationales are evaluated as **YAGNI**: welcome when the dropped work was genuinely unneeded, rejected when they bypass the floor under a YAGNI label. This floor does not override `cp-abstract` (≥3 use cases before abstraction) or `cp-understand` (understand before changing).
- **Match injection mechanism to content type** — `tools: [...]` description for declarative capabilities (state, availability); `skills/<name>/SKILL.md` for on-demand workflow (L2, body loads only when relevant); `ctx.session.hook("context")` injection only for imperative policy or protocol the model must internalize. Never duplicate tool capability in fixed system-prompt text; never put a workflow guide in a fixed prompt when a skill can carry it. Decision matrix and OCP examples: `DEVELOPING.md` §"Plugin authoring — injection mechanism".

---

## Cost Red Line — Model Pricing

**Hard cap on every model referenced in shipped configs (per M tokens): input ≤ $3.00, output ≤ $15.00.** Applies to `profiles/**`, `providers/**`, `opencode.template.jsonc`, and any other file that names a `provider/model` ref. Violating models are excluded regardless of capability — e.g. `gpt-6-astra` ($10/$50) and `claude-opus-5` ($5/$25) breach the cap and must not be referenced. Price source: `models.dev` provider catalogs. Scope: the cap binds metered per-token picks; flat-rate subscriptions (coding/token plans) and local router gateways (`codex-router`, `claude-code-router`, `omniroute`, `qoder-router`, `llm-router`, `antigravity-router`) have no per-token price in models.dev — their picks are bounded by plan inclusion and gateway availability, not by this numeric cap (but a profile referencing them must not imply API pricing). When adding or re-tiering a profile, verify both rates before committing; an over-cap pick is a build failure. The cap bounds spend, not capability: within the cap, always pick the strongest model for the tier's job — `max` feeds advisor/architect and deep/final `code-review` (review quality is paramount; family profiles must use the family's strongest reviewer even when a cheaper cross-family rival exists), while routine review triage uses the pro-tier `code-review-fast`; `pro` should track the vendor's latest code-specialized model. Price-capability sanity source: the LLM Price–Capability Kill Line (https://mappedinfo.github.io/llm-price-kill-line/) — prefer kill-line frontier survivors; never reference models flagged deprecated there.

---

## Ships vs. Dev-Only

- **Ships** (injected into OpenCode agent system prompts): `instructions/*.md`, `prompts/*.md`, `skills/*/SKILL.md`, plus all files under `plugins/`, `profiles/`, `providers/`. These are the actual prompts users consume — every line costs tokens on every session, forever.
- **Dev-only** (repo tooling, never shipped): `AGENTS.md`, `DEVELOPING.md`, `scripts/`, `docs/`, `install/`, `bin/`. These guide contributors working on this repository and are never injected into a user's OpenCode session.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kenlin8827/opencode-prime](https://github.com/kenlin8827/opencode-prime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
