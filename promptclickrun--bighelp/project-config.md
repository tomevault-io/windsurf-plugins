---
trigger: always_on
description: bighelp is a Swift 6 SwiftUI client for personal AI agents running through Hermes. This repository contains:
---

# bighelp repository instructions

## Product and repository

bighelp is a Swift 6 SwiftUI client for personal AI agents running through Hermes. This repository contains:

- the iOS and iPadOS app under `Bighelp/`;
- a notification service extension under `BighelpNotificationService/`;
- a Live Activity extension under `BighelpLiveActivity/`;
- shared extension models under `BighelpActivityShared/` and `Bighelp/NotificationShared/`;
- Swift Testing and XCUITest targets under `BighelpTests/` and `BighelpUITests/`;
- (the Hermes plugin lives in its own repository, promptclickrun/bighelp-plugin).

Production uses independently authenticated native Hermes REST and `/api/ws`.
Cloud services are allowed only for optional notifications and Live Activities. Never
restore Link chat routing, queues, workers, account-catalog startup gates, or the
old paired Direct listener. See `docs/NATIVE_TRANSPORT.md`.

Read these before making material changes:

- `README.md`
- `docs/ARCHITECTURE.md`
- `docs/DEVELOPMENT.md`
- `CONTRIBUTING.md`
- `bighelp-plugin/README.md`, `bighelp-plugin/PROTOCOL.md`, and `bighelp-plugin/SECURITY.md` for plugin or protocol work

## Hermes is authoritative

Hermes is authoritative for agent execution, sessions, transcripts, tools, approvals, policy, profiles, projects, scheduled tasks, models, reasoning configuration, and lifecycle state. bighelp is a native client and platform integration, not a second Hermes runtime.

The app may keep bounded local presentation and offline state, but it must reconcile remote behavior with Hermes. Optimistic UI is never proof that Hermes accepted or committed an operation. Request, session, agent, profile, revision, sequence, acknowledgement, and authorization coordinates must match before state is committed.

Use the current official Hermes documentation and source when a Hermes capability or contract is material:

https://hermes-agent.nousresearch.com/docs

Relevant native surfaces include documented:

- Python plugin registration APIs;
- platform adapters and `BasePlatformAdapter` behavior;
- plugin hooks, tools, skills, slash commands, and CLI subcommands;
- approval transports and platform actions;
- authenticated plugin API namespaces;
- native media extraction, authorization, and delivery APIs;
- profile, project, session, config, model, cron, and policy APIs;
- TUI gateway JSON-RPC methods and event streams;
- the API server only where its documented feature set fits the requirement.

Choose the narrowest public Hermes surface that preserves Hermes ownership. Verify the current contract rather than inferring support from an old implementation or copied code.

## Forbidden Hermes integration shapes

Do not solve a missing capability by building or depending on:

- a shim protocol that imitates a Hermes API;
- a sidecar service or daemon that becomes required for normal bighelp operation;
- a patch, fork-only behavior, or edit to an installed Hermes checkout such as `~/.hermes/hermes-agent`;
- monkeypatching, `sys.modules` replacement, runtime method replacement, or mutation of private Hermes globals in production code;
- a copied agent loop, session manager, approval engine, scheduler, policy engine, project registry, profile store, transcript store, or model catalog;
- a shadow database that competes with Hermes for authoritative runtime state;
- a parallel MCP client, provider client, or gateway control plane when Hermes already owns that connection and exposes a supported route;
- shelling out to emulate an in-process Hermes API when a documented plugin or TUI-gateway method exists;
- a compatibility fallback that changes authority, weakens authentication, broadens approval scope, or produces behavior newer Hermes versions would reject.

Tests may patch or stub imports to isolate behavior. Production code may not use those test techniques as architecture.

If current Hermes native support cannot express a required outcome, stop that implementation path. Record the exact user outcome, missing public capability, current surfaces examined, security and compatibility requirements, and the smallest upstream Hermes extension that would close the gap. Change the bighelp design or contribute upstream. Do not conceal the gap behind local infrastructure.

## Current architecture

- `BighelpAppComposition` is the composition root for production and fixture dependencies.
- `ShellFeatureStore` owns route-scoped feature-model creation and lifetime.
- Mutable feature models are generally `@MainActor` and `@Observable`.
- Features use narrow client protocols so production and deterministic fixture implementations share the same UI and model logic.
- Keep transport wire models separate from app domain and UI models.
- Validate, normalize, and bound every untrusted value before converting it to application state.
- Production composition selects `NativeWorkspaceRuntime` through `WorkspaceConnectionStore`; `Bighelp/DirectHermes` owns native transport. Retained Link chat implementations are disabled compatibility code, not a fallback.
- Hermes is the remote session authority. Local session records are a cache plus local draft/presentation state.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [promptclickrun/bighelp](https://github.com/promptclickrun/bighelp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
