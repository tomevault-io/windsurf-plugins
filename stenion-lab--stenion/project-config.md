---
trigger: always_on
description: This is the internal index for working _in_ this codebase. It holds the rules and conventions that
---

# Stenion — Working notes for Claude Code

This is the internal index for working _in_ this codebase. It holds the rules and conventions that
must not be broken, and points to the public docs for everything else. **Don't duplicate substance
here** — each public doc owns its content:

- **[`README.md`](README.md)** — what Stenion is, the pitch, local quick-start.
- **[`ARCHITECTURE.md`](ARCHITECTURE.md)** — monorepo layout, what each package does, data flow, deploy.
- **[`METHODOLOGY.md`](METHODOLOGY.md)** — the source of truth for every factor's formula, thresholds, weights.
- **[`API.md`](API.md)** — the public API contract as a consumer meets it: endpoints, live example
  responses, the `ok`/`failed` union, staleness, rate limits, errors. Rendered at `/docs/api`.
- **[`CONTRIBUTING.md`](CONTRIBUTING.md)** — how to write an adapter, conventions, PR expectations.
- **[`ROADMAP.md`](ROADMAP.md)** — what's live, what's planned, what's out of scope, and open taxonomy questions.

## What this is (one line)

An open-source, live risk-intelligence platform for Stellar/Soroban DeFi lending protocols
(Blend + Kinetic shipped). The differentiator is **continuous, on-chain-derived risk scoring** — not
TVL tracking. Full framing in [`README.md`](README.md).

## Non-negotiable rules

These override any default behavior and are enforced in code and review:

- **Payment must never affect the score.** Protocols pay for visibility/speed/private tooling —
  never for a better number. The real registry is always free, public, ranked purely on score. Paid
  "Spotlight" is a visually separate, clearly-labeled section.
- **AI only explains/summarizes real underlying data** — never an independent risk assessment.
- **Adapters read trustless on-chain data** (Soroban RPC + Horizon) — never self-reported figures.
- **No fabricated numbers.** When real data isn't available for a factor, use a clearly-flagged
  neutral baseline (e.g. `adminKeySafety`'s contract-admin `60`) — never an invented value.
- **`API.md`'s example responses are captured live, never written from the types.** A doc written
  from `db/src/store.ts` reproduces the type rather than the truth, and what a client observes is
  not always what a route sets (the CDN eats `s-maxage`). Re-`curl` them when a shape changes.
- **Code and `METHODOLOGY.md` are not allowed to drift.** Any change to a formula/threshold/weight
  changes both together, at the same review bar. Shared rulebook logic that two adapters would
  otherwise duplicate lives in [`core/src/scoring.ts`](core/src/scoring.ts), so it can't drift
  between them.
- **A scoring change that makes old scores non-comparable bumps `METHODOLOGY_VERSION`**
  (`core/src/types.ts`), stamped onto every run by the indexer. History is never backfilled —
  `risk_scores` stores only outputs, not the raw inputs — so the discontinuity is labeled, not
  hidden.
- **Findings are not scores.** Verifiable observations we can't or won't grade go in the protocol
  page's Findings section (`dashboard/app/lib/protocol-notes.ts`), never into a factor. Nothing
  there is read by any scoring path.
- **An unscored listing is a coverage statement, never a score.** Protocols we assessed and don't
  score are published on the registry from `dashboard/app/lib/coverage.ts` — never in the
  `protocols` table, whose never-scored state (`safetyScore: null`, "never run") means _our pipeline
  hasn't got there yet_ and must not be collided with a deliberate decision. Nothing in that section
  renders a numeral, so "not scored" can't be misread as "scored badly"; it's unranked, because
  ranking what we didn't score is meaningless. Every entry needs a protocol-specific reason, a
  one-sentence `summary` for its registry row, and a `verify` sentence, and any claim resting on a
  reading (a balance) needs an `asOf` — figures we never checked against contracts are not a source.
  Enforced in `coverage.test.ts`. Each entry's full reasoning lives at **`/coverage/<id>`**, served
  only from that module — never `/protocol/<id>`, which would either 404 in the API while rendering
  in the dashboard or force `getProtocolDetail` to serve two shapes.
- **Nothing unscored may sit inside a ranked ordering, and a position numeral means a position.**
  The registry's sort/filter/search is pure functions in `dashboard/app/lib/registry-query.ts`
  (state in query params, never component state) so this is testable rather than a rendering habit.
  Score sorts rank the scored set only; unscored entries are a separate block below, and the
  never-scored `safetyScore: null` rows are a third block of their own. Name sort is the sole
  ordering allowed to merge them, because alphabetical asserts no ranking. The `#` column renders
  **only** under score-descending and is removed — not blanked — otherwise: under score-ascending
  "01" would label the lowest score as first. Enforced in `registry-query.test.ts`.
- **A registry entry is a market, not necessarily a protocol — and it must say which.** An entry
  running another protocol's contracts (the YieldBlox pool on Blend V2) carries
  `ProtocolMetadata.deployedOn`, published as `deployedOn` on both API responses and rendered

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stenion-lab/stenion](https://github.com/stenion-lab/stenion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
