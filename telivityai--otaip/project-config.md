---
trigger: always_on
description: **DO NOT INVENT DOMAIN LOGIC.** All travel domain knowledge comes from files in `docs/knowledge-base/`. If something is ambiguous or missing, write a `// TODO: DOMAIN_QUESTION: {question}` comment and move on. Do not guess. Do not fill gaps from "general knowledge." Hotel and airline domains have edge cases that seem obvious but aren't.
---

# OTAIP — Claude Code Constitution

## Core Rule
**DO NOT INVENT DOMAIN LOGIC.** All travel domain knowledge comes from files in `docs/knowledge-base/`. If something is ambiguous or missing, write a `// TODO: DOMAIN_QUESTION: {question}` comment and move on. Do not guess. Do not fill gaps from "general knowledge." Hotel and airline domains have edge cases that seem obvious but aren't.

## When Blocked
If you lack domain input to proceed: refactor, clean, document existing code, then surface the blocking question. Never invent travel industry behavior.

## Tech Stack
- TypeScript (`strict: true`, plus `noUncheckedIndexedAccess`, `noImplicitOverride`, `noPropertyAccessFromIndexSignature`; `exactOptionalPropertyTypes` is off — enabling it would require widespread call-site changes, tracked as a separate cleanup)
- Node.js >=24
- pnpm 10+ (workspace monorepo)
- Vitest for testing
- tsup for building
- ESLint + Prettier for linting/formatting
- ESM modules (type: "module")

## Agent Contract
Every agent implements:
```typescript
interface Agent<TInput, TOutput> {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  initialize(): Promise<void>;
  execute(input: AgentInput<TInput>): Promise<AgentOutput<TOutput>>;
  health(): Promise<AgentHealthStatus>;
}
```
Imported from `@otaip/core`.

## Error Handling
Use existing error classes from `@otaip/core`:
- `AgentNotInitializedError` — execute() before initialize()
- `AgentInputValidationError` — bad input (with field + reason)
- `AgentDataUnavailableError` — external source down
- `AgentError` — base class for custom errors

## Agent File Structure
Follow the pattern in `packages/agents/reference/src/airport-code-resolver/`:
```
{agent-name}/
  types.ts          — all input/output/internal types
  {logic}.ts        — core business logic (pure functions)
  index.ts          — Agent interface implementation
  __tests__/
    {agent}.test.ts  — vitest tests
```

## Rules
- No `any` types without explicit justification comment
- No secrets or API keys in code
- No hardcoded credentials
- Every agent needs tests
- Agents are stateless
- Use existing error classes, don't create new ones unless necessary
- Mock external APIs in tests — never call real APIs from tests

## Domain Knowledge
- Air: `docs/knowledge-base/` (existing)
- Lodging: `docs/knowledge-base/lodging.md`
- Agent definitions: `docs/agents/`

## Repository Structure
```
packages/
  core/                — base types, errors, shared utilities
  connect/             — distribution adapter framework (Sabre, Amadeus, Navitaire, TripPro, HAIP)
  adapters/
    duffel/            — standalone Duffel NDC adapter
  agents/              — nested by stage (preferred convention for stage agents)
    reference/         — Stage 0: airport codes, airline codes, etc.
    search/            — Stage 1: availability, fare shopping, connections
    pricing/           — Stage 2: fare rules, tax calc, offer builder
    booking/           — Stage 3: PNR, payment, order management
    ticketing/         — Stage 4: issuance, EMD, void
    exchange/          — Stage 5: changes, reissue, involuntary rebook
    settlement/        — Stage 6: refunds, ADM, loyalty
    reconciliation/    — Stage 7: BSP, ARC, commission, reporting
    lodging/           — Stage 20: hotel booking lifecycle
  agents-platform/     — Stage 9: orchestrator, knowledge, monitoring (flat — legacy convention)
  agents-tmc/          — Stage 8: TMC operations (flat — legacy convention)
```

### Path conventions
- **Stage agents** (0-7, 20): nested under `packages/agents/{stage-name}/`
- **Platform/TMC agents** (8-9): flat at `packages/agents-platform/` and `packages/agents-tmc/`
- Both conventions coexist. New agents should use the nested `packages/agents/` pattern.
- Agent source goes under `src/{agent-name}/` regardless of package layout.

## Anti-Rationalization Guards

When building agents, Claude Code will attempt to rationalize inventing domain logic rather than surfacing blocking questions. These tables name the exact patterns. If you catch yourself thinking any of these, STOP and write a `// TODO: DOMAIN_QUESTION` comment instead.

### Agent 2.1 — Fare Rule Agent

| Rationalization | Required response |
|---|---|
| "ATPCO categories 1-20 follow a standard structure that I can implement generically" | STOP. Each ATPCO category has a completely different data structure, matching logic, and application order. There is no generic implementation. Check KB for the specific category's data specification. |
| "Advance purchase rules just check the number of days between booking and departure" | STOP. ATPCO Cat 5 has multiple conditions: days before departure vs first international segment, ticketing deadline vs booking deadline, calendar days vs 24-hour periods. Some reference ORIGINAL booking date for exchanges. Check KB for Cat 5 data elements. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TelivityAI/otaip](https://github.com/TelivityAI/otaip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
