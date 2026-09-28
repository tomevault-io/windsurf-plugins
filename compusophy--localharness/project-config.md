---
trigger: always_on
description: Project context for Codex sessions. Read this first.
---

# AGENTS.md

Project context for Codex sessions. Read this first.

> Keep under **40K chars** (harness cap). A *map + gotchas*, not a reference —
> facet semantics in `contracts/README.md`, wire detail in
> `examples/tempo_tx_live.rs`, agent detail in `web/llms.txt`. Adding a fact?
> Cut or compress an older one; don't append.

## What this is

`localharness` is a Rust-native, **model-agnostic** agent SDK **and** a
self-sovereign browser-resident agent platform built on it. ONE crate. `cargo add`
gives an agent loop with streaming text, tool calling, hooks, policies, triggers,
MCP, and context compaction (behind a `Connection`/`ConnectionStrategy` seam —
Gemini/Anthropic/OpenAI/Mock backends ship). Build with `browser-app` on wasm32 and
you also get the live IDE at `<name>.localharness.xyz`.

- [crates.io/crates/localharness](https://crates.io/crates/localharness) (version: `Cargo.toml` / crates.io — never pinned here) · [github.com/compusophy/localharness](https://github.com/compusophy/localharness)
- Native: stable Rust 1.85+, tokio. wasm32: same crate, browser.
- Live: `localharness.xyz` (apex) + wildcard `*.localharness.xyz` (per-user agents).
- On-chain: EIP-2535 Diamond on Tempo Moderato testnet (chain 42431, RPC
  `https://rpc.moderato.tempo.xyz`); Tempo MAINNET live (chain 4217) — flip via
  `mainnet`. See **Canonical addresses** below.

## Canonical addresses

Live in ONE place — `src/registry/chain.rs`: `MAINNET` (chain 4217, RPC
`https://rpc.tempo.xyz` — the LIVE default) and `MODERATO` (testnet, chain 42431,
RPC `https://rpc.moderato.tempo.xyz` — explicit dev opt-in), each pinning diamond
/ `$LH` token / fee_token / explorer. Contract architecture + facet detail:
`contracts/README.md`. Don't hand-copy addresses into docs — the table that used
to live here went stale; READ chain.rs.

**Per-facet addresses are NOT pinned** — facets churn via `diamondCut`; query live
via DiamondLoupeFacet (`facets()` / `facetAddress(selector)`). The diamond address
is the only durable handle.

## Repo layout

> Subsystems own NESTED `AGENTS.md` specs (auto-loaded when an agent works in that
> dir; update the matching one when you change a subsystem): `src/app` (UI / overlay
> modals / no-DOM / one-box-input), `src/registry` (on-chain gas/tx/relay),
> `src/backends` (provider wire quirks), `src/bin/localharness` (CLI), `src/rustlite`
> (cartridge compiler), `src/filesystem` (FS impls + at-rest encryption /
> EXEMPT_FILES), `src/builtins` (tool schema-lint rule), `src/soliditylite`
> (EVM-subset compiler), `src/bashlite` (sandboxed shell — fuel + confirm-gate),
> `src/connections` (L3 transport seam — wasm cfg-gating), `proxy/` (the
> separate-deploy credit proxy / relay / metering), `contracts/` (Diamond
> cut/storage/deploy gotchas; facet semantics in contracts/README.md), `web/`
> (cache-buster + cartridge-worker↔Rust parity), and `scripts/` (release atomicity
> + PS5 trap + QA tooling). Every `src/` module dir + proxy + contracts + web +
> scripts own a spec; the root stays a whole-repo MAP and detail lives in the specs.

```
src/                  library crate
├── lib.rs            re-exports + module roots
├── agent.rs          Agent facade (L1): start_gemini/start_anthropic/start_mock
├── conversation.rs   Conversation + ChatResponse (L2)
├── connections/      Connection / ConnectionStrategy traits (L3)
├── content.rs        Content, Media, Part (user message types)
├── tools.rs          Tool trait + ToolRunner + ClosureTool
├── hooks.rs          6 hook traits + HookRunner
├── policy.rs         Predicate / Policy / Decision + workspace_only
├── triggers.rs       Trigger trait + TriggerRunner + every()
├── runtime.rs        cfg-gated spawn + sleep_ms + MaybeSendSync marker
├── encoding.rs       THE canonical hex/address/amount codecs (don't re-roll one)
├── turn_flow.rs      pure turn-classification + MAX_AUTO_CONTINUATIONS (hoisted
│                     from app::chat so its loop-guard tests run natively); a
│                     text-only turn continues ONLY while a plan has open steps
├── plan.rs           pure update_plan checklist core; an OPEN plan = turn_flow
│                     treats text-only turns as mid-plan, not goodbye (#75/#69/
│                     #67). Live copy: app/chat/plan_state.rs
├── session_prompt.rs the in-tab base prompt as a PURE fn (app-or-test gated);
│                     fact-pins guard telemetry-earned lines + a SIZE BUDGET
│                     makes growth deliberate. app/chat/prompt.rs = thin wrapper
├── agent_tools.rs    THE canonical tool list (AGENT_TOOLS) — ungated so the docs
│                     gen AND the browser allowlist grid share it (docs_manifest
│                     is wallet+native-only). Never fork a 2nd list
├── turn_stage.rs     pure stage state machine for the pending-turn "paying →
│                     thinking → streaming" line (painted by app/chat/stage.rs)
├── router.rs         pure INTENT-ROUTER core: exact-allowlist free/metered cost
│                     gate ahead of the metered chat turn (balance/UI/docs-FAQ
│                     answered free; '!' forces metered; IntentClassifier trait =
│                     the local-Gemma seam; wired in app/chat/router_wire.rs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compusophy/localharness](https://github.com/compusophy/localharness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
