---
trigger: always_on
description: This file is loaded when an agent is dispatched to work on a cloister
---

# AGENTS.md — agent dispatch + persona guide for cloister

This file is loaded when an agent is dispatched to work on a cloister
bead. It complements [CLAUDE.md](CLAUDE.md) (which is the project memory
for any Claude session); this file is specifically about working
*through the rsry bead pipeline*.

## Personas relevant to cloister

| Agent | When dispatched |
|---|---|
| `dev-agent` | Default for `feature` and `task` issue types. Implements; commits; comments. |
| `scoping-agent` | Reads the bead description first; if scope is unclear, narrows or splits before implementing. Best for beads that touch >2 files or cross subsystem boundaries. |
| `architect-agent` | Owns `design` issue type. Drafts ADRs, decomposes them into implementation beads, files dependency edges. Does not implement. |
| `staging-agent` | Owns `review` issue type. Adversarial reviewer. Does not modify code; files findings as comments or sub-beads. |
| `pm-agent` | Strategic / cross-repo. Surfaces overlap, abandoned experiments, scope creep. Read-only by default. |
| `janitor-agent` | Cleanup. Dead-session detection, worktree garbage, stale branch sweep. Cluster-level concerns. |

The bead's `owner` field assigns the agent. `rsry_bead_create` infers
from `issue_type` if `owner` is omitted.

## Active decade

`interlace-substrate` is the current workstream. Run
`rsry_thread_list --decade interlace-substrate` to see threads:

| Thread | Purpose |
|---|---|
| `adrs` | Decision documents (ADR-0007/0008/0009/0010/0011/0012) |
| `identity-lease` | notme lease minter, WASM verifier, lease middleware, leyline-sign wasm32 emit. **Substrate shipped 2026-05-09** (cloister-bd7770 / -9d49eb). Wiring into `mcp.ts` is `cloister-b89fdb`. |
| `discovery` | `.well-known/interlace/` + capabilities surface |
| `attestation` | `peer_attestations` in **TrustStore** (per ADR-0012; was BeadStore-resident before the 2026-05-09 correction). Cross-DO writes use ADR-0003 content-addressed handoff. Gated on `cloister-960f68` (BlobStore Phase 1 hardening). |
| `deployment` | CF Tunnel / WARP off-platform story; "cluster in a pod" via workerd v8 proc-iso (`cloister-be0607`, ADR-0009 implementation) |
| `oss-prep` | CLAUDE.md / AGENTS.md / CI workflows / README+ARCH sync; ll-open 0.2.0 sync (`cloister-bd8c41`) |
| `vault` | Lift `notme/vault/` → `cloister/vault/` (AGPL-3) + cross-repo notme cleanup |
| `audit` | (now empty — audit findings folded into surface threads for parallelizability) |

## What changed 2026-05-09 — start your map here

If you're a fresh agent, read these first; they are the corrections
that supersede earlier in-flight assumptions:

- **TrustStore is a separate DO** from BeadStore. Singleton per cluster
  at `idFromName("cluster")`; hypervisor-layer per ADR-0011's three-
  criterion test. Bead state stays in BeadStore (per-repo). See ADR-0012.
- **Cross-DO writes use content-addressed handoff** (ADR-0003 phase 1).
  Workerd ACID is per-DO; `bead_create → BlobStore → BeadStore →
  TrustStore` walks four steps, but BlobStore is idempotent so failure
  recovery is well-defined. The bolded ADR-0007:154 "same SQL transaction"
  rule is replaced by this multi-step but recoverable pattern.
- **Lease middleware substrate is real and wired**. `verifyAndUpsertLease`
  runs the full pipeline (header parse → wasm32 cert verify → claims
  required → epoch + window → Web Crypto Ed25519 sig → scope →
  TrustStore RPC). Wired into `McpEdgeRoute.handlePost` per
  `cloister-b89fdb` (closed). Active when `INTERLACE_ROOT_PUBKEY` is
  set; skipped in dev/test when unset (deployment-binding granularity,
  NOT per-request bypass).
- **CredentialVault is wired** as a hypervisor-tier singleton DO
  (`env.VAULT_STORE`) per ADR-0013 (slice-grant enforcement via V8
  isolate + service-binding-as-syscall). Envelope encryption + per-
  credential `allowedSubs` glob enforcement; plaintext credential bytes
  never cross the RPC boundary. Library lifted from notme/vault
  (cloister-9ad9eb); DO wrapper at `src/vault-store.ts`. Open: in-
  cluster bundle identity propagation (gated on the first workerd-
  bundle Worker — see `cloister-ac30e7`).
- **`leyline-sign` lives upstream in LLO** at `rs/ll-open/sign/` as of
  2026-07-09 (bead ley-line-open-7226e3 / LLO PR #160; cloister-side
  deletion `cloister-8f4d3f`). Cloister pulls it via a git dep pinned
  by SHA in `rs/crates/cas/Cargo.toml`; the wasm build output
  (`rs/target/wasm32-unknown-unknown/release/leyline_sign.wasm`)
  stays at the same path. Historical: the crate was lifted from
  agentic-research/ley-line 2026-05-08/09 as `rs/crates/sign/`, then
  reconsolidated back upstream once ll-open's `leyline-sign` proved
  the canonical home per ADR-0035.
- **Threat model is the contract** for completing the lease/attestation
  arc. See `docs/security/threat-model.md` (math-friend authored,
  `cloister-bd32b1`).

## Bead lifecycle on cloister

1. **Pick up** — `rsry_bead_search` first, `rsry_dispatch` once you've
   confirmed scope.
2. **Implement** in the worktree at `~/.rsry/worktrees/cloister/<bead-id>/`.
   Run `pnpm install` first; export `CLOISTER_SCHEMA_ROOT` if the bead
   touches the manifest schema (see CLAUDE.md "Working in worktrees").

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentic-research/cloister](https://github.com/agentic-research/cloister) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
