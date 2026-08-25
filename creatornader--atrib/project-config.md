---
trigger: always_on
description: atrib makes the actions an AI agent takes provable. Every agent action becomes a signed, chain-linked record committed to a public Merkle log. Downstream consumers (the agent itself, merchants, auditors, other agents) can independently verify what happened without trusting any operator.
---

# atrib: the protocol for verifiable agent actions

## What this is

atrib makes the actions an AI agent takes provable. Every agent action becomes a signed, chain-linked record committed to a public Merkle log. Downstream consumers (the agent itself, merchants, auditors, other agents) can independently verify what happened without trusting any operator.

Product teams can put that record in the action path. A host can check an action before it runs, and later work can carry the signed context forward. atrib records tool calls (MCP via `@atrib/mcp-wrap`, framework-native via `@atrib/agent` or `@atrib/openinference`), transactions (six payment protocols detected per the [payments profile](docs/payments-profile.md): ACP, UCP, x402, MPP, AP2, a2a-x402), and the agent's own intentional records and reads through the two cognitive verbs: `attest` (write: observation by default, annotation via `ref.kind: "annotates"`, revision via `ref.kind: "revises"`) and `recall` (read: shape-dispatched lookups, lineage walks, and handoff verification). The seven legacy primitive names (`emit`, `atrib-annotate`, `atrib-revise`, the `recall_*` family, `trace`, `summarize`, `atrib-verify`) stay mounted as permanent aliases over the same handlers per [D164](DECISIONS.md#d164-attestrecall-verb-rename-and-primitive-surface-collapse); records are byte-identical either way.

The same substrate supports agent recall, third-party audit, commerce settlement, handoffs, and controlled high-impact actions when a harness or runtime puts atrib records in the action path. atrib sits between identity (DIF/W3C) and payment rails (ACP/UCP/x402/MPP/AP2). It also makes post-hoc claims about agent activity provable.

When explaining the graph, keep the
[D118](DECISIONS.md#d118-primary-trace-path-is-a-presentation-rule-over-trace-and-chain)
framing stable: atrib has one graph with two reading planes. The chronology
plane records event history and continuity. The declared-relationship plane
records signed claims about which records informed, anchored, annotated, or
revised other records. `/v1/chain` and `/v1/trace` are projections over that
graph, and the explorer primary trace path is a product presentation rule over
both projections.

The canonical protocol identity used across the README top, spec abstract, root
package metadata, and GitHub repo description:

- **Headline:** Verifiable agent actions.
- **Sub-line:** Every action becomes signed context for the next.
- **Tagline:** Agents that reason from a past they can prove.

The GitHub repo description adds the concrete proof mechanism: `Ed25519
signatures, Merkle log proofs, independently verifiable by anyone.`

Product docs and README sections can use the commercial frame when they explain
host integrations. In that context, atrib records can sit in the action path, so
a host can check actions before they run and later sessions, agents, teams,
organizations, and protocols can verify what carried forward. Do not put that
commercial framing in the spec abstract or GitHub repo description unless the
protocol identity itself changes.

When a public contribution needs to distinguish the commercial app from this
open substrate, call the app `atrib` and this substrate `atrib protocol`.
`atrib Workflows` is historical product language, not a forward-facing product
name. Keep commercial-app claims within their proved boundary.

The complete protocol specification is in `atrib-spec.md` ([§0](atrib-spec.md#0-foundations)-[§7](atrib-spec.md#7-harness-integration-patterns)). The technical architecture overview is in `ARCHITECTURE.md`. Read the spec before making any implementation decisions.

## Repository structure

```
atrib/
  .github/
    scripts/check-log-smoke.mjs # Shared public log smoke checker for deploy and scheduled workflows. Checks response shape plus TTFB/total latency budgets for pubkey, stats, recent, feed, explorer shell, and public website/explorer asset parity.
  README.md                    # Public-facing project description (developer entry point)
  STATUS.md                    # Implementation status: deployed services, published/deprecated package census, protocol surface implemented vs spec-defined-only. README carries the summary; this file is the enumeration.
  CLAUDE.md                    # THIS FILE: hub doc, conventions, invariants
  DESIGN.md                    # Product design system source of truth: current state, target state, tokens, components, surface backlog
  atrib-spec.md                # The single source of truth for the protocol

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [creatornader/atrib](https://github.com/creatornader/atrib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
