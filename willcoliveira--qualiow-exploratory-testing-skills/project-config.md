---
trigger: always_on
description: AI-powered exploratory testing — Claude acts as a Principal QA Engineer across three
---

# Qualiow Exploratory Testing Skills

AI-powered exploratory testing — Claude acts as a Principal QA Engineer across three
surfaces: web applications, mobile apps on a simulator/emulator, and backend/API/infra
acceptance criteria that never reach a screen. It explores, finds bugs, and writes
structured reports.

Package: `qualiow-exploratory-testing` (npm) · plugin name `qualiow` · MIT · Node >= 22.4.

## How It Works

No framework and no test scripts — Claude's own QA reasoning plus a driver per surface.

- **Web** — `playwright-cli` (`@playwright/cli`, a hard dependency) for browser control.
  `/qa-explore` and `/qa-explore-quick`.
- **Mobile** — `bin/mobile-cli.mjs` (called through `bin/mcli`), a Maestro / `xcrun simctl` /
  `adb` shim that mirrors the `playwright-cli` command surface. `/qa-explore-mobile`.
- **Backend / API / infra** — `git show` on the implementation branch, read-only `aws-cli`
  probes, and `fetch` from inside the authenticated page. `/qa-verify-backend`.

## Skills

| Skill | Purpose |
|-------|---------|
| `/qa-explore` | Full exploratory testing session (45 min, 8 phases) |
| `/qa-explore-quick` | Quick focused session on a single page or feature (15 min) |
| `/qa-explore-mobile` | Session on a simulator/emulator — native apps OR web apps in the real device browser (iOS Safari / Android Chrome), via mobile-cli; mode selected by the target config |
| `/qa-verify-backend` | Verify backend/API/infra acceptance criteria with no UI surface — branch review, read-only cloud probes, direct API probes, AC traceability matrix |
| `/qa-gather` | Gather and analyze requirements from files, URLs, tickets or text into a session context file |
| `/qa-explore-report` | Generate, regenerate or reformat a report from an existing session |
| `/qa-explore-feedback` | Post-session feedback capture (false positives, missed bugs) |
| `/qa-explore-cleanup` | Session cleanup and archival |
| `/qa-knowledge-add` | Add new heuristics, techniques, docs to the knowledge base |
| `/qa-knowledge-list` | Browse the knowledge base |
| `/qa-target-setup` | Configure a target application (auth, scope, domain) |

One sub-agent ships with them: `.claude/agents/qa-gather-agent.md`, the background runner
behind `/qa-gather`.

Under a plugin install the same skills are namespaced: `/qualiow:qa-explore` etc.

## Usage

```bash
# Explore a public site
/qa-explore https://testers.ai/testing/

# Explore with a saved target config
/qa-explore --target company-staging

# Quick check on a specific page
/qa-explore-quick https://app.example.com/checkout

# Verify a backend ticket against its ACs
/qa-verify-backend --target local-my-service --context output/context/TICKET-123-context.md

# Add new QA knowledge
/qa-knowledge-add
```

## Project Structure

- `.claude/skills/` — the 11 skills, **canonical**; `skills/` is the generated mirror shipped
  to npm and used by the plugin (`npm run sync:plugin`; CI fails on drift — never hand-edit
  the mirror)
- `.claude/agents/` — `qa-gather-agent`; mirrored to `agents/` the same way
- `.claude-plugin/plugin.json` — Claude Code plugin manifest
- `bin/` — `mcli` + `mobile-cli.mjs` (mobile driver), `wadb`, `wk-ios` + `wkeval.mjs` (iOS
  WebKit DOM bridge), `setup-mobile.sh`, `doctor-mobile.sh`
- `scripts/` — repo dev tooling, not shipped: `kb-sync.mjs` (rebuild/check the knowledge
  manifest), `check-pack.mjs` (assert the npm tarball contents), `sync-version.mjs` (keep
  `.claude-plugin/plugin.json` in step with `package.json`)
- `data/knowledge/` — YAML knowledge base (heuristics, techniques, checklists) versioned as
  releases v0.1.0–v0.6.0, 29 entries, indexed by `manifest.yml`
- `data/domains/` — domain profiles, **YAML only** (`*.yml`; the `.md` variants were removed
  in 2.0.0)
- `data/templates/` — bug report, session report, charter, coverage map, AC probe matrix,
  expected-behaviour spec
- `data/targets/` — target application configs (URL, auth, scope, domain)
- `data/security/SECURITY-POLICY.md` — long-form security policy
- `docs/` — GETTING-STARTED, MOBILE-SETUP, BACKEND-VERIFICATION (shipped); KNOWN-ISSUES,
  ARCHITECTURE-DECISIONS (repo-only)
- `src/` — TypeScript CLI (`cli/`), report formatters (`formatters/`), Zod schemas
  (`schemas/`), utilities (`utils/`: redaction, session parsing, paths, metrics, validation)
- `tests/` — vitest unit tests and fixtures
- `output/sessions/`, `output/bugs/`, `output/context/` — session outputs, the aggregated bug
  list, and gathered ticket context
- `qa/` — project-local config in a consumer project: `target.yml`, `.env`, `bin/`
- `.auth/` — Playwright storage-state files **and** persistent browser profile directories
  holding live session cookies (gitignored in full; the API lane depends on them)

## Configuration Resolution

- Target: `--target <name>` → `qa/target.yml` → `data/targets/_default.yml`
- Credentials: `qa/.env` → `.env`, referenced by **variable name** in YAML, never by value.
  The names the shipped configs use: `QA_USER`, `QA_PASS`, `QA_TOKEN`, `QA_AWS_PROFILE`,
  `QA_AWS_REGION`, `QA_API_TOKEN`, `EXAMPLE_API_KEY`
- Output always under `<cwd>/output/`
- Full rules: `.claude/skills/qa-explore/references/paths.md`

## Session Output


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [willcoliveira/qualiow-exploratory-testing-skills](https://github.com/willcoliveira/qualiow-exploratory-testing-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
