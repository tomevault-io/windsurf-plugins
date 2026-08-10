---
trigger: always_on
description: This project is building a platform for "vibe coded" personal applications and AI agents that run inside a strong sandbox.
---

This project is building a platform for "vibe coded" personal applications and AI agents that run inside a strong sandbox.

The following files are commonly important to reference:

* packages/workshop-shared/node_modules/capnweb/README.md: Explains how to use Cap'n Web RPC, which is used extensively for client-server communications.
* packages/workshop-shared/src/api.ts: Defines the RPC API used between the frontend and backend.

The project structure is:

* packages/workshop-frontend: The Gadgets Workshop UI.
    * This is a pure single-page app, running entirely client-side.
    * It speaks to the backend using an RPC API over a persistent WebSocket connection.
    * Uses React, Kumo UI (https://kumo-ui.com/api/component-registry), Phosphor icons, and Vite.
* packages/workshop-backend: The Gadgets Workshop server.
    * Runs on Cloudflare Workers.
    * This is the **kernel**: it defines the architecture and is held to a higher bar than UI/gatekeeper code. Reviewers read *every line* of `workshop-backend` and of API changes in `workshop-shared`, so keep diffs here small and elegant. Concretely: doc-comment **every** exported member of the `workshop-shared` public API (types, consts, and functions — not just interfaces); never introduce a hand-written interface that mirrors an RPC interface plus an `as unknown as` cast (derive from the real type instead, or rethink the design); and prefer reusing existing mechanisms over adding parallel ones. Capability-based security note: a resource becomes "ambient" (auto-injected) only by user/admin configuration — a gatekeeper must never assert its own ambience. When a change to this package is large, split it by concern into separate PRs (and at minimum group commits so `workshop-backend`/`workshop-shared` can be reviewed apart from UI), since fewer kernel lines = easier review.
    * `format-blueprints/` holds the **output format** blueprints the deployment ships with, committed as data: a `<name>.gadget` archive plus a `<name>.json` sidecar giving its `blueprintId`, prose, and `output` presentation. `scripts/build-format-blueprints.mjs` globs that directory (override with `FORMAT_BLUEPRINTS_DIR`, which lets a fork ship its own set without touching this submodule) into the gitignored `src/generated/format-blueprints.ts`, so `build`, `types:check` and `test` all run the generator first. Replace one with `pnpm import:format-blueprint <export.gadget> <blueprintId>`, or add one with `pnpm import:format-blueprint <export.gadget> --new <name>`; never edit a `blueprintId` after deploy, since the install and promotion are keyed on it and a rename orphans the old entry. See `format-blueprints/README.md`.
* packages/workshop-shared: Shared API definitions between client and server.
    * This defines the application's RPC interface.
    * The RPC protocol is Cap'n Web, which has similar semantics to Cloudflare's Worker-to-Worker RPC system, while being able to run in a browser over WebSocket. Read the readme for details.
* packages/configurator-ui: Type-only component helpers used by optional gatekeeper resource configurator UI modules.
    * Gatekeeper configurator UI modules are compiled by `scripts/build-gatekeeper-configurator.mjs` as part of package builds.
* packages/gatekeeper-*: Gatekeeper workers for external service integrations.
    * Each gatekeeper runs as a separate Cloudflare Worker.
    * Gatekeepers handle OAuth flows and provide sandboxed access to external APIs.
    * A gatekeeper may declare `VendorDescription.autoProvisionsAccount`: it can mint a connected account with no OAuth flow (via `GatekeeperVendor.createAccount()`, which takes no user identity). For such gatekeepers the deployment admin picks a per-vendor mode in the admin Gatekeepers panel — **disabled** / **optional** / **enabled** (default **optional**) — resolved in `provisioning-policy.ts`: `enabled` auto-provisions the account for every user (forced, and hidden from the Connectors list), `optional` lets each user opt in from the Connectors page, and `disabled` offers it to no one (existing accounts go dormant). The Workshop persists the account in the user DO like any connected account (the account capability — not an asserted identity — is the authority thereafter). The **account** (a `GatekeeperUser`) declares in its `AccountDescription` whether it provides an agent **singleton** (`singleton: { tsType }`) and/or a **management UI** (`providesUi`). The Workshop auto-provides the singleton to the owner's workspaces as an **ambient gatekeeper record**, folded into each chat's env as a **named chat binding** (named by the gatekeeper's `suggestedBindingName`; see `prepareChatBindings` in overseer.ts) that the agent reads in `executeCode` (`getSession`/`getAgentCatalog`), each read recorded as an observation. It is not bound to any gadget by default — most gadgets never call it programmatically — but the agent may wire it into a gadget's binding list with `setGadgetBinding` when the gadget's persistent code needs it. The UI is hosted at `/gatekeepers/$appId` (the gatekeeper's vendor id, e.g. `/gatekeepers/context`) via `startAppUi({ isAdmin })`. The two are orthogonal — an account can declare either, both, or neither.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/cloudflare-os](https://github.com/cloudflare/cloudflare-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
