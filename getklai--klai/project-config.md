---
trigger: always_on
description: Model-neutral rules for every agent in this monorepo (Codex, Claude, or other).
---

# Klai — Agent Operating Rules

Model-neutral rules for every agent in this monorepo (Codex, Claude, or other).
Codex reads this file natively. Claude reads it via `@AGENTS.md` in `CLAUDE.md`.
A nested `AGENTS.md` closer to the file you edit overrides anything here.

> Enforcement note: Claude runs Stop/PreToolUse hooks that mechanically enforce
> some of these rules. **Codex runs no such hooks** — under Codex these rules
> are self-enforced only. Treat them as MUST, not as suggestions.
> Codex-specific notes: `.agents/codex/README.md`.

## Prime directive — autonomy on execution, strictness on claims

- Keep going. Do not ask permission to proceed, to push approved work, or to
  move to the next step. Stop only for (a) a real decision the user must make,
  or (b) a failing guard: CI red, test failure, or unverifiable external state.
- You may NOT claim "fixed / done / deployed" without evidence per claim.
  "Looks correct" / "should work" / "reviewed the code" is not evidence — it
  scores zero. Prove it, don't assert it.

## Always-on engineering discipline

- **data-before-code** — Trace real logs / DB / runtime before fixing. No
  guessing, no stacked patches. For production: query VictoriaLogs by
  `request_id:<uuid>`. One root cause confirmed by data = one fix.
- **fail loudly** — No silent fallback on external-provider drift. Unknown
  external state = raise an error or report explicit residual risk. No
  "best-effort success" when the core mutation failed. (Database-layer RLS
  defense-in-depth is deliberate and stays — this rule is about app-layer
  shims, not the DB security model.)
- **minimal changes** — Only what was asked. No drive-by refactors, reformatting,
  or "improvements" to untouched files.
- **clean over clever, no parallel old+new** — Remove the code your change
  replaces in the SAME change: no dead fields, no commented-out blocks, no old
  and new flow living side by side. Clean solutions over defensive clutter. (This
  is about removing what you replaced — not editing untouched files; it composes
  with "minimal changes", it does not contradict it.)
- **scale the answer to the problem** — Lead with the simplest solution that
  works (the 5-minute fix if one exists); escalate to a bigger design only when
  the problem demands it. No SPEC for something that affects 1–5 people. State
  explicitly what you deliberately did NOT do.
- **verify-changes-landed** — Before reporting done: `git diff --stat` (right
  files?), service health/logs (running new code?), and a Playwright
  click-through for any UI change (real user flow works?).
- **search broadly when changing a default/name** — grep every consumer, all
  case variants (kebab, snake, camel, Pascal, SCREAMING_SNAKE). Defaults have
  unbounded blast radius.
- **no plausible assumptions** — Do not infer `message.sources`, Zitadel
  password policy, streaming chunks, BFF cookies, OIDC flows, or performance
  paths from intuition. Require evidence from code, tests, logs, docs, or an
  explicit user confirmation.

## Codex + Serena

Codex only auto-loads `AGENTS.md` files. It does not automatically read
`.serena/project.yml`, `.serena/memories/**`, or `.claude/rules/**`.

- For code exploration under Codex, load Serena first when the Serena MCP tools
  are available: call `initial_instructions`, then follow the project prompt in
  `.serena/project.yml`.
- Use Serena for source-code symbol discovery and edits. Use `rg`/normal file
  reads for Markdown, YAML, config, env examples, and other non-code files.
- If Serena is unavailable, continue with local source inspection and state that
  residual risk in the final answer.

Serena memory files in this public repo are public documentation. Keep them
evergreen and contributor-safe only: repo layout, coding patterns, public
service contracts, local development, and self-hosting templates are allowed.
Do not write Klai production hostnames, SSH aliases, IPs, tunnel topology,
secret names that are not already part of public code/config contracts,
operator runbooks, business/GTM plans, compliance records, or customer context
to `.serena/memories/**`. Production operations belong in the private
`klai-infra` repo; business, GTM, compliance, and research context belongs in
the private `klai-private` repo.

## Local / Production Browser Testing Contract

Before any browser-driven portal check (Playwright, Browser MCP, manual
localhost navigation, screenshots, or E2E), establish which runtime contract is
being tested. Do not guess ports, auth mode, proxy target, or whether a
localhost listener belongs to this workspace.

- **Local standalone UI** means: frontend in `VITE_AUTH_DEV_MODE=true`, backend
  in `AUTH_DEV_MODE=true`, frontend proxying to the local backend, no Zitadel,
  no production login redirect. The required preflight is:
  `scripts/local-dev-status.sh --mode local --strict`. If it fails, fix setup or
  report the failure. Do not continue clicking through login.
- **Production E2E** means: no localhost target. Validate credentials/target
  with `scripts/local-dev-status.sh --mode prod-e2e`, then run from
  `klai-portal/frontend` with `source .env.local && npm run test:e2e:prod`.
- **Conductor ports**: if `CONDUCTOR_PORT` is set, the frontend port is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GetKlai/klai](https://github.com/GetKlai/klai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
