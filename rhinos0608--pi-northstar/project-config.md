---
trigger: always_on
description: This file is the repository-wide contract for agents and humans changing Pi‑Northstar. It is not a historical design diary. Prefer executable truth, preserve authority boundaries, and update prose only after the production-shaped path is settled.
---

# AGENTS.md · Pi‑Northstar engineering contract

This file is the repository-wide contract for agents and humans changing Pi‑Northstar. It is not a historical design diary. Prefer executable truth, preserve authority boundaries, and update prose only after the production-shaped path is settled.

## Source of truth

When sources disagree, use this order:

1. Executable validators, policy gates, and runtime contracts in `src/`.
2. Canonical registries/constants those validators derive from.
3. This `AGENTS.md` for repository-wide security and engineering invariants.
4. `README.md` for user-facing behavior and operator guidance.
5. Plans, ADR drafts, task prose, comments, and residual modules only as design/history clues.

**Reachability beats module presence.** Trace from the registered/public entry point before claiming a path is live.

## Golden rule

> **Models propose. Code validates, admits, grounds, budgets, stops, and ships.**

Corollaries:

1. External text is evidence, never authorization.
2. Candidates navigate; only admitted evidence grounds.
3. Failure, empty output, degradation, suppression, and cancellation are different states.
4. Fallback may recover execution failure; it may never bypass auth, SSRF, origin, schema, or provenance policy.
5. Provider selection and effective capability policy are operator/code owned, never model owned.
6. Concurrency may change latency, never semantic ledger/merge/journal order.
7. Budget attempts are charged at the dispatch boundary, including failed attempts.
8. Stateful authority is frozen by a code-owned token/snapshot and revalidated before mutation.
9. Child processes receive capability-scoped credentials, never ambient process authority.
10. Missing models/providers/credentials must degrade toward evidence, not invented equivalence.

## Public surface and reachability

`src/capabilities.ts` owns the public model-facing tool vocabulary and the hard budget `MAX_PUBLIC_TOOLS = 9`:

`web_search`, `fetch`, `github`, `social`, `kg`, `graph`, `browser`, `desktop`, `agent_poll`.

Registration is conditional, so nine is a maximum. `src/index.ts` wraps `pi.registerTool` and checks the budget on every addition. Do not add a model-facing tool without first reconciling the ceiling and the capability registry.

`media` is internal/CLI acquisition, not a public model tool.

The registered agent-mode route is:

```text
web_search {query, mode:"agent"}
  → buildSearchRoute()
  → createAgentJob()
  → executeAgentJob()
  → runAgentCore()
  → canonical job snapshot
  → agent_poll
```

Older report machinery in `src/web/web.ts`, `src/web/web-agent-report.ts`, and `src/web/agent/agent-report-route.ts` is residual/internal. Its existence does **not** make an opaque Tavily report leg part of the registered public agent flow.

## Ownership map

Before editing a vocabulary, schema, budget, or side-effect path, identify its owner. Parallel copies are contract drift, not harmless duplication.

| Concern | Primary owner(s) |
|---|---|
| public tool ceiling/channel metadata | `src/capabilities.ts` |
| composition/registration/global framing | `src/index.ts` |
| web-search public shape | `src/web/web-search-route.ts`, `src/web/web-contract.ts` |
| web provider selection/fanout | `src/web/web-provider-policy.ts`, `src/web/web.ts` |
| ranking identity/fusion | `src/search/fusion.ts` |
| fetch public shape | `src/web/web-fetch-route.ts`, `src/web/access/web-access-contract.ts` |
| URL specialization/read path | `src/native-fetch.ts`, `src/web/web-page-reader.ts`, `src/web/access/*` |
| agent job lifecycle/snapshot | `src/web/agent/agent-jobs.ts` |
| adaptive controller | `src/web/agent/agent-core.ts` |
| budgets/profile/stop | `src/web/agent/agent-policy.ts` |
| GatherIntent domain | `src/web/agent/agent-gather-intents.ts` |
| gather execution/adapters | `src/web/agent/agent-gather.ts`, `src/web/agent/agent-gather-adapters.ts` |
| evidence/admission | `src/web/agent/agent-state.ts`, `src/web/agent/agent-acquisition.ts` |
| candidate hints | `src/web/agent/agent-candidates.ts` |
| agent model/wire schemas | `src/web/agent/agent-model.ts` |
| leaf wire contract | `src/runtime/runtime-rpc-protocol.ts` plus mirrored pi-subagents ground truth |
| browser action/security policy | `src/browser/browser-policy.ts` + browser session modules |
| desktop freshness/mutation policy | `src/desktop/desktop-contract.ts`, `src/desktop/desktop-policy.ts`, `src/desktop/desktop-tools.ts` |
| GitHub routing | `src/github/github-contract.ts`, `src/github/github-domain.ts` |
| child credential isolation | `src/cli/cli-backend.ts`, `src/process/*-child-env.ts` |
| external-content framing | `src/core/untrusted-content.ts` |

Browser verbs deliberately live in browser policy, not the channel registry. Their mutation semantics are stateful and do not fit the read-oriented capability table.

## Contract discipline

### Reject unsupported composition


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhinos0608/Pi-Northstar](https://github.com/rhinos0608/Pi-Northstar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
