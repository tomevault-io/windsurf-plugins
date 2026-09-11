---
trigger: always_on
description: This file applies to the entire repository. Read the component guide and the
---

# Maple monorepo agent guide

This file applies to the entire repository. Read the component guide and the
matching skill under `.agents/skills/` before changing that component. The
current source and tests take precedence over historical design documents.

## Repository map

- `apps/maple-research/`: the existing React/Vite/Tauri Maple application,
  including desktop Agent Mode. Read its [guide](apps/maple-research/AGENTS.md)
  for runtime placement, native security, and exact-app validation.
- `apps/maple-agent/`: GPUI desktop-v2 prototype, ACP and proxy CLI. Read its
  [guide](apps/maple-agent/AGENTS.md) and `$develop-maple-agent`. Its runtime and
  update discovery are separate from Research and its existing Agent Mode.
- `sdk/`: TypeScript/React and Rust Maple SDKs for the OpenSecret backend. Read
  `$develop-opensecret-sdk` and the SDK documentation.
- `proxy/`: the separate OpenAI-compatible relay, also consumed by the desktop
  app. Read `$develop-maple-proxy` and the proxy documentation.
- `services/updates/`: the updater Worker. Preserve its deployed identity,
  public endpoints, installed-client compatibility, and verified metadata flow.
- `services/opensecret/`: the OpenSecret Rust backend, including its pinned
  toolchain, local operator recipes, and signed PCR files. Read its
  [guide](services/opensecret/AGENTS.md) and `$develop-opensecret`.
- `docs/`, `scripts/`, `.github/`, `.agents/`, `flake.nix`, `flake.lock`,
  `justfile`, and `repo.meta.json`: shared documentation, tooling, CI, and
  repository identity. Keep active workflows and discoverable skills at root.

For SDK consumption, follow the [consumer version policy](docs/sdk-publishing.md#consumer-version-policy):
prefer independently pinned published versions, allow local links during active
development, and review the actual SDK source before a client release. SDK
publication and upgrading a consumer are separate decisions.

The backend import does not change TEE deployment or introduce EIF publication
through GitHub Actions. Preserve the manual signed-PCR compatibility procedure
in [the backend guide](services/opensecret/docs/pcr-compatibility.md) and the
existing `OpenSecretCloud/opensecret` raw URLs used by installed clients.

## Start safely

1. Confirm the checkout, branch, and worktree state. Preserve unrelated user
   changes and do not switch branches or rewrite history in a dirty checkout.
2. Read relevant source and tests before proposing placement. Prefer the
   narrowest existing component; write down cross-component API contracts
   before changing them.
3. Before creating configuration or starting or stopping services, determine
   whether an external development environment owns ignored environment files,
   the root `.local/tauri-workspace.json` overlay, ports, or processes. Preserve
   those resources and follow that environment's lifecycle instructions.
4. Use the versions and platform dependencies pinned by the owning Nix flake.
   Run shared app recipes from the repository root; enter component directories
   only where their command instructions require it. Do not add substitute
   global toolchain bootstraps to docs or CI.
5. Use `just clean-local` for the Research checkout. Raw `cargo clean` may erase
   a shared Nix Cargo build directory used by other checkouts. Follow the
   owning component's cleanup contract for other components.

## Shared security and compatibility

- Keep authentication, authorization, encrypted persistence, provider
  credentials/routing, model policy, and inference-usage truth in OpenSecret.
  Client UI, feature flags, and billing presentation are not authority.
- Never log access/refresh tokens, API keys, plaintext prompts/responses,
  decrypted payloads, or credential-bearing headers/environments. Sanitize
  errors before exposing them across a trust boundary.
- Treat network, renderer, model, file, deep-link, and tool input as untrusted.
  Validate at the boundary that owns the privileged effect. Preserve account
  isolation, cancellation, lifecycle ownership, and fail-closed permissions.
- Keep secrets out of `VITE_*`; these are public build-time values. Preserve
  local ignored configuration and do not overwrite it with defaults.
- Preserve shipped application identity, signing/updater contracts, public
  APIs, and protocol compatibility unless the user authorizes the change.
  TypeScript and Rust SDK transports, retry behavior, and API coverage differ;
  verify every affected client path when a backend contract changes.
- Follow nearest code, error, accessibility, and test patterns. Avoid unrelated
  dependency upgrades, rewrites, and generated-file edits. Use the appropriate
  generators and inspect their deltas; never hide them with Git index flags.

Load `$review-maple-security` for auth, proxy, Agent tooling, native
capabilities, filesystem/process access, deep links, or persistence work. Keep
findings in the task review or another authorized destination; keep durable
standards and methodology in guides and skills.

## Validation and publication authority

Start focused, then run complete gates for the layers changed. The app's
[validation guide](apps/maple-research/AGENTS.md#validation-is-proportional-evidence)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MaplePrivacyLabs/Maple](https://github.com/MaplePrivacyLabs/Maple) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
