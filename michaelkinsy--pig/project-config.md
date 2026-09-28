---
trigger: always_on
description: Operating rules for AI agents maintaining `pig`, the Go port of upstream `pi`. This file is context, not human marketing. Keep it small, current, and enforceable.
---

# AGENTS.md

Operating rules for AI agents maintaining `pig`, the Go port of upstream `pi`. This file is context, not human marketing. Keep it small, current, and enforceable.

## Mission

`pig` must match upstream `pi` observable behavior unless `DIVERGENCES.md` records a numbered, scrutinized exception. Treat the port as a compiler from upstream TypeScript behavior to Go behavior: `PORT_MAP.md` defines the file map, parity scenarios define behavior, `parity/coverage.md` reports proof, and gates keep the claims honest.

Source of truth:
- `.upstream/current/` is the upstream mirror.
- `coding/pigversion/pigversion.go` pins the upstream version (re-exported by `coding/upstream.go` as `coding.UpstreamVersion`).
- `PORT_MAP.md` maps upstream files to Go files, deferred entries, or designed-out entries.
- `parity/scenarios/<family>/*.toml` define observed behavior.
- `parity/interfaces/upstream-v<version>.json` is the compiler-derived public
  package interface denominator; observable adapter inventories such as
  `cli-v<version>.json` extend it for non-package user contracts;
  `mapping-v<version>.json` records each Pig disposition and closure evidence. Generated inventory drift and strict mapping
  are Foundation gates. PORT_MAP remains the file navigation roll-up until
  Foundation I finishes generating its statuses from these ledgers.
- `parity/upstream-sync/v<version>.toml` accounts for every changed tracked source file in a version leap.
- `parity/async-contracts.toml` accounts for Promise/async semantics across the complete pinned upstream source tree.
- `parity/coverage.md` and the generated block below report verification.
- `DIVERGENCES.md` records intentional differences.

No prose status claim overrides those files.

When landing a new port, record its full upstream source path in a production
`// Ports packages/.../file.ts` comment and update PORT_MAP in the same change.
The drift gate rejects explicit port claims left not-started, deferred, or n/a.
Use partial until behavioral closure is reviewed; source existence is not proof.

## Product-extension boundary

Stock PiG contains upstream-parity behavior and the inert, product-neutral mechanisms needed to discover, validate, select, build, verify, load, isolate, and run Packages, Resources, extensions, Piglets, and Piglet Binaries. A Piglet selects and activates product behavior. PiG Standard owns branded presentation, opinionated defaults, selected extensions, onboarding, games, and other optional product workflows.

Classify every proposed additive capability before implementation:

1. `required substrate` remains in Stock PiG.
2. `inert capability` remains in Stock PiG only when an extension or Piglet must select it.
3. Composition, presentation, policy, and branded behavior belong in PiG Standard or another explicit Piglet or Package.
4. Caller-free or unneeded behavior is deleted.
5. Behavior that overlaps Pi but differs observably is fixed or recorded in `DIVERGENCES.md` after explicit approval.

Use `product-neutral diagnostics` for generic reports that describe Stock mechanisms. Use `explicit unsafe opt-in` only for a capability that stays off by default, is visible at configuration boundaries, and must not be selected by PiG Standard.

A Piglet cannot supply the parser, resolver, verifier, SDK bridge, isolation boundary, or build machinery needed to load itself. Do not move those bootstrap mechanisms into PiG Standard. Do not activate Standard behavior in Stock PiG. Do not put product UI, Piglet-specific recommendations, workflow-specific logic, or linked extension products under `internal/` or `cmd/` except for the minimum generic host capability.

Additive records use `docs/additive-features.md`. Observable Stock PiG differences use `DIVERGENCES.md`. Place one short typed source marker at each production decision point. Tests do not satisfy the source-marker requirement.

## Pig-owned format and update policy

Pig-owned source, manifest, record, graph/plan, builder, Marketplace,
publication, and subprocess formats have one strict current unversioned shape
while Pig and its clients upgrade atomically. Release/content identity, exact
upstream-owned schema versions, and external-standard versions remain. Do not
add a Pig-owned format discriminator, negotiation, compatibility reader,
migration, redirect, or dual write unless exact upstream regularly versions the
equivalent schema or the user explicitly approves independently deployed
compatibility.

Self-update preserves Pi's command routing but uses one proven native ownership
tier per operation. Standalone download, package-manager, immutable
Piglet Binary, OCI/Image, and unsupported/read-only paths must
not substitute for one another after work starts. Downloads require complete
checksum and overflow-safe size verification; startup checks remain bounded and
best-effort, while explicit update failures remain actionable.

## Extension API porting rules

The pig extension system is a **port** of the upstream pi extension API,
not a replacement for it. Two downstream-only constructs exist so we can
stay subprocess-only without giving up the pi extension API:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MichaelKinsy/PiG](https://github.com/MichaelKinsy/PiG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
