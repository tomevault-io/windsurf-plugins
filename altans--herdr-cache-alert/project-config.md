---
trigger: always_on
description: **Cache Alert** (repo `AltanS/herdr-cache-alert`) — a [Herdr](https://herdr.dev) plugin that puts a
---

# CLAUDE.md — working agreement for this repo

**Cache Alert** (repo `AltanS/herdr-cache-alert`) — a [Herdr](https://herdr.dev) plugin that puts a
prompt-cache countdown on every agent pane and marks the turns that missed the cache. Plugin id
`herdr.cache-alert` (manifest: `herdr-plugin.toml`). Bun + TypeScript, no build step. Orientation:
[`README.md`](./README.md).

## The claim contract — MANDATORY

**No bare cache constant may enter `src/`.** Every TTL, minimum-token count and cost multiplier is a
`Sourced<T>` (see [`src/claims.ts`](./src/claims.ts)) carrying a value, a confidence, a
documentation URL, the ISO date it was checked, and a verbatim quote from that page.

When you add or change one:

1. **Fetch the page.** Not a search result, not a memory, not another agent's summary. If you cannot
   open it, you cannot cite it.
2. **Quote verbatim.** If the sentence cannot be quoted, the claim is not `documented` — downgrade it
   to `reported` or `inferred` and say in `note` exactly what you could and could not confirm.
3. **Stamp `retrievedAt` with the real date.** This is the load-bearing field; `claims --stale`
   exists entirely to make it rot loudly.
4. **Record gaps as gaps.** `codex.*` carries an explicit "OpenAI publishes no Codex-specific TTL"
   note. An honest hole is worth far more than a plausible number, because the plugin's whole value
   is that its numbers can be trusted.

`confidence: "observed"` is reserved for something MEASURED on this machine from the harness's own
telemetry. It outranks every documented rule. Never use it for anything you reasoned your way to.

## Versioning — MANDATORY

Cache Alert is **SemVer**ed, and the version is **enforced**, so it never silently drifts.

**The version lives in two files that must always agree, plus a matching CHANGELOG entry:**
`herdr-plugin.toml` (canonical — Herdr reads it) · `package.json` · newest `## [x.y.z]` heading in
`CHANGELOG.md`.

**Before committing any functional change** (anything under `src/`, `scripts/`, `bin/`, or the
manifest) you MUST:

1. **Bump** the version in both files to the same number. The axis is **what the operator has to
   do**, not how visible the change is:
   - **PATCH** (`0.2.0 → 0.2.1`): the code now does what it was always meant to do — bug fixes and
     internal refactors. Re-verifying a claim's `retrievedAt` is a patch.
   - **MINOR** (`0.2.0 → 0.3.0`): something is there that wasn't — a new adapter, command, action,
     or config option. Existing setups keep working untouched.
   - **MAJOR** (`0.2.0 → 1.0.0`): the operator must change something — a config key or CLI flag
     renamed or removed, a stored-format break, an adapter contract change that breaks third-party
     adapters.
2. **Add a `CHANGELOG.md` entry** under a new `## [x.y.z] - YYYY-MM-DD` heading (Added / Changed /
   Fixed). Use the real date. **Style: crisp and short** — one line per change, no prose paragraphs.
3. **Run `scripts/check-version.sh`** — it must print `✓`.

Doc-only changes (`*.md`) don't need a bump.

**Tag the release when you push it.** `git tag -a vX.Y.Z -m "Cache Alert X.Y.Z" && git push --follow-tags`
so the tag ships *with* the release. One `v<x.y.z>` tag per shipped version on the remote.

## Build / run

- **No build step.** Plugin panes run `scripts/run.sh <entrypoint>`, which picks a runtime and
  executes `src/<entrypoint>.ts` directly. Herdr launches plugin commands with a minimal
  environment — never assume anything is on `PATH` there; the shim checks the usual install
  locations too.
- **Both runtimes are supported and both must keep working.** Bun is preferred when present,
  Node ≥ 22.6 otherwise. Two rules keep that true:
  **(1)** every runtime difference goes in `src/runtime.ts` — no `Bun.*` or Node-only global may
  appear anywhere else in `src/`; **(2)** `erasableSyntaxOnly` is on, because Node *strips* types
  rather than compiling them — no enums, namespaces, or constructor parameter properties.
  Check both after touching either: `CACHE_ALERT_RUNTIME=node ./bin/herdr-cache-alert status` and
  `CACHE_ALERT_RUNTIME=bun …`.
- **Three gates, all must pass:** `bun run lint` (oxlint + the vendored anti-slop
  rules in `tools/oxlint/`, `--max-warnings 0`), `bun x tsc --noEmit`
  (TypeScript 7, strict, with `noUnusedLocals/Parameters` and
  `noUncheckedIndexedAccess`), and `bun run test`.
- **The suite is Node's built-in runner** (`node:test`), so it adds no dependency
  and there is still no build step. `scripts/test.sh` points `HERDR_PLUGIN_STATE_DIR`
  and `HERDR_CONFIG_PATH` at a throwaway directory — without that a test writes
  the operator's real `state.json` and lands on a live pane's countdown. It then
  runs the CLI under BOTH runtimes, which is the part `node:test` cannot cover.
- **Test the seams that have already shipped a bug.** Every rule in the list below
  is a test now: precedence order, the claim contract applied to the rules that
  actually ship, the six token names, the event payload's spelling, `sessionKey`.
  A test whose name states the reason is worth more than one that states the
  input — the bug was never that the code did the wrong thing on purpose.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AltanS/herdr-cache-alert](https://github.com/AltanS/herdr-cache-alert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
