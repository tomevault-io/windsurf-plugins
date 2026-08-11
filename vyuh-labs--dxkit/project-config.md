---
trigger: always_on
description: Every external tool (cloc, gitleaks, semgrep, graphify, jscpd, ruff, etc.) MUST be:
---

# CLAUDE.md — DXKit Development Rules

## Architecture Rules

### 1. Tool invocation goes through the registry

Every external tool (cloc, gitleaks, semgrep, graphify, jscpd, ruff, etc.) MUST be:

- **Defined** in `src/analyzers/tools/tool-registry.ts` (TOOL_DEFS)
- **Detected** via `findTool(TOOL_DEFS.xxx, cwd)` — never hardcode binary paths
- **Installed** via `vyuh-dxkit tools install` — never ad-hoc npx/pip calls

Builtins (grep, find, wc, git, node) are exempt — they're always available.

**Executed downloads are checksum-verified (4.0.3 / T2.1).** Any network
artifact an install command fetches goes through
`dxkit_fetch <url> <sha256> <dest>` — defined ONCE in
`src/analyzers/tools/install-exec.ts` (the single install executor prepends
it), with the pinned hash declared in the registry beside the URL. It fails
CLOSED on mismatch. The arch-check bans a raw curl/wget-with-URL in
`tool-registry.ts` and an `Invoke-WebRequest` without a `Get-FileHash` check
(annotate `// unverified-download-ok` for a justified exception);
`test/tool-registry-version-pins.test.ts` pins that every `dxkit_fetch` call
carries a 64-hex sha and that no unverified fetch exists in any install
command. Bumping a tool version means re-pinning its hash (prefer the
publisher's published checksum file where one exists — gitleaks, codeql).

**CI tool discoverability is registry-derived, never a hardcoded PATH.** The
places a tool binary can live are declared once — `getSystemPaths()` plus each
tool's `probePaths` in the registry — and `findTool` probes them. The CI PATH
export (`exportToolPathsToGithubEnv`, run at the end of `tools install`) reads
the SAME sources and writes them to `$GITHUB_PATH`, so the per-language dep audit
finds its native scanner (osv-scanner / pip-audit / govulncheck / cargo-audit)
in a workflow step instead of silently falling back to a wrong-artifact scanner.
A new language pack that installs a scanner to a new directory declares it in the
tool's `probePaths` (already required for detection) and is thereby covered in CI
automatically — do NOT hardcode a per-ecosystem bin dir in a workflow template.
`test/tool-paths-ci.test.ts` pins that every tool `probePath` appears in the
export.

### 2. Never duplicate tool invocation logic

Each tool has ONE gather function (e.g., `gatherGraphifyMetrics` in `tools/graphify.ts`).
If another module needs that tool's output, it MUST call the existing function.
Do NOT rewrite the command string, JSON parsing, or error handling in a new file.

**Bad**: Copy-pasting the graphify Python script into parallel.ts
**Good**: Calling `gatherGraphifyMetrics()` from parallel.ts

#### One concept, one code path (2.30 — the recurring-bug fix)

The most persistent dogfood-bug class is a special case of Rule 2: **one concept
is computed in two independent code paths, a fix lands in the path you're
editing, and the sibling keeps misbehaving.** Real instances: the `env-in-git`
metric count vs its per-finding producer (a fix to exempt `.env.example` reached
only the count); a placeholder-secret filter added to the gitleaks provider but
not the `grep-secrets` fallback; `flow.stripUrlPrefixes` threaded into the map +
gate gathers but not the diagnose + detect gathers. Each shipped because the
duplicate path was in a different file/layer and nothing forced a single entry
point.

When a concept has multiple consumers, give it ONE entry point and route every
consumer through it (canonical examples added in 2.30):

- committed env-file detection → `trackedEnvFiles` in
  `src/analyzers/security/env-files.ts` (the only `git ls-files .env`);
- a repo's flow model WITH its policy config applied → `gatherRepoFlowModel` in
  `src/analyzers/flow/gather.ts` (loads `stripUrlPrefixes` / `specs` itself, so a
  surface cannot forget them); the raw `gatherFlowModel` is only for
  explicit-config callers (the two-ref gate, cross-repo publish, the map CLI);
- a repo's declared MODEL SET with its policy config applied →
  `gatherRepoModelSet` in `src/analyzers/model-schema/gather.ts` (loads
  `schema.specs` itself; the raw `gatherModelSet` is only for the two-ref
  gate's explicit-config sides). What schema DRIFT a diff contains →
  `diffModelSets` in `src/analyzers/model-schema/model.ts`, the ONE drift
  computation consumed by BOTH the guardrail gate and `schema diff`
  (pinned by `test/schema-gate-diff-parity.test.ts` — same
  parity-test discipline as the flow gate/join pair below);
- whether a consumed `(method, path)` is SERVED → `servedMatch` /
  `catchAllPrefixCovers` in `src/analyzers/flow/model.ts`. BOTH the join (doctor's
  `diagnoseFlow`) and the integration gate (`evaluateFlowGate`) resolve a call
  against the served set through this ONE catch-all-aware predicate. The recurring
  shape here (the flagship instance of the _semantic_-divergence variant): the gate
  held a LOSSY projection of the concept — a `Set<string>` of exact `${method}
${path}` keys, which discards catch-all structure — so it did exact membership
  only and hard-blocked every call served by a `[...slug]` / `/**` catch-all that
  doctor resolved cleanly. `buildServedMatcher` rebuilds the catch-all prefixes
  from the key set so the gate inherits the join's resolution. The consumed side's

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vyuh-labs/dxkit](https://github.com/vyuh-labs/dxkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
