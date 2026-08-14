---
trigger: always_on
description: > Read this file at the start of EVERY session before touching code.
---

# AGENTS.md — Invariants

> Read this file at the start of EVERY session before touching code.
> These are the load-bearing rules. Breaking one silently is how this project
> rots across sessions. If you must break one, it’s a deliberate, isolated
> increment with its own ADR in DECISIONS.md — never a side effect.

-----

## Session ritual

**START**

1. Read this file.
1. `make test && make demo-<previous-increment>` — the baseline must be green
   before you change anything.
1. State out loud the single increment ID you are building this session. One
   increment per session.

**END**

1. New `make demo-<this-increment>` passes.
1. Full `make test` green (every prior demo still passes).
1. If you touched the prompt prefix, re-run the prefix-hash test.
1. Append exactly 3 lines to PROGRESS.md: shipped / stubbed / next.

Do not start the next increment in the same session. A half-finished second
increment is the exact failure this whole structure exists to prevent.

-----

## The architecture in one paragraph

We are building a security/provenance/memory layer **around** oh-my-pi (omp),
not from scratch. The harness is **TypeScript on Bun, in-process with omp**. The
**only** Python is `scanner-sidecar/` (the pure Unicode scanner). We extend omp
through hooks, custom tools, and its SDK. See DECISIONS.md ADR-0001.

-----

## Writing style

- **Never use em dashes** (the long dash, Unicode U+2014). Use commas, periods,
  colons, or hyphens instead. This applies everywhere: code, comments, UI strings,
  commit messages, ADRs, and PROGRESS.md.

-----

## Invariants (do not violate)

### 1. Extend omp; never fork it.

Prefer a hook or custom tool over any change that would require forking omp.
omp ships hundreds of releases; a fork is permanent merge pain. If you believe
something genuinely cannot be done via hook/tool/SDK, STOP and write an ADR
before forking anything.

### 2. The language boundary is fixed.

The harness is TypeScript. The ONLY directory containing Python is
`scanner-sidecar/`. Never add a `.py` file anywhere else. Never reimplement the
scanner in TypeScript. If a second Python surface seems necessary, that’s a
drift signal — stop and write an ADR.

### 3. Fail-closed is law.

Any failure to obtain a valid scan result — sidecar dead, malformed response,
timeout, missing id — MUST be treated as “block / quarantine,” never “safe.”
No code path may treat “scan unavailable” as “pass.” There is a test that kills
the sidecar mid-run and asserts the gate blocks; it must stay green forever.

### 4. The quarantine gate runs in-process.

The security gate is a `pre` hook inside omp’s runtime. It must not depend on a
network call or a fragile boundary to do its blocking. The scanner (pure, behind
the sidecar) may be out-of-process; the GATE that acts on its output may not.

### 5. Untrusted content is always delimited and always late.

All user-provided, retrieved, imported, or externally stored text enters prompts
only inside `UNTRUSTED_CONTENT_START` / `UNTRUSTED_CONTENT_END`, only after
scanning + sanitation, and only AFTER the cache breakpoint (never in the frozen
prefix). The system prompt instructs the model to treat delimited content as
data, never instructions.

### 6. The prompt prefix is frozen and byte-stable.

Prompt layers 1–4 (identity/safety, tool-use/permission policy, stable coding
rules, security/trust-boundary rules) are byte-identical across all requests.
Volatile context — date, cwd, git branch/status, env — that omp auto-injects
MUST live in the tail, after the cache breakpoint. Putting volatile bytes in the
prefix busts the KV cache every turn. Verify with the prefix-hash test.

### 7. Trust labels are a closed set.

Exactly: `trusted | untrusted | suspicious | quarantined`. No other values.

### 8. Events use exact names.

Every logged event uses a name from the `EventName` enum in `contracts.ts`.
Emitting an unknown event name must raise. Every event carries `run_id`,
`session_id`, and `artifact_id` when an artifact is in scope.

### 9. Stable IDs everywhere.

Every run, finding, and approval has a stable string `*_id`. Reuse omp’s
Snowflake session/run IDs; mint your own for findings/approvals. IDs are never
regenerated for the same logical entity.

### 10. DuckDB schema freezes on first write.

Schema changes only ever happen through numbered migration files. Never edit a
table definition in place. The schema is a frozen contract.

### 11. UI text is never cramped into wrapping columns.

Readability is load-bearing. Any list/grid of rows (providers, models, settings,
tiles, chips) MUST keep each row's primary label on one line: the label takes the
space and ellipsizes on overflow (`white-space: nowrap; overflow: hidden;
text-overflow: ellipsis`); it NEVER mid-word wraps inside a narrow column.
Secondary chips/badges are `white-space: nowrap`. Grid tracks use a generous
`minmax()` (about 220px or more) so labels do not fold; when content will not fit,
use fewer/wider columns or a single column, NEVER a fixed narrow multi-column grid
that word-wraps its text into unreadable slivers. If a label can be long, widen the
column or truncate it with a tooltip. This applies to every panel, popup, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mlcyclops/lucidagentide](https://github.com/mlcyclops/lucidagentide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
