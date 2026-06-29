---
trigger: always_on
description: provides zero enforcement. If the file is not written to disk, the gate does not exist.
---

# Copilot Instructions

<!-- ForgeCraft | 2026-05-04 | tags: UNIVERSAL, LIBRARY | npx forgecraft-mcp refresh . to update -->

## Project Identity
- **Repo**: https://github.com/K-Arthur/opencode-harness
- **Primary Language**: typescript
- **Domain**: library
- **Sensitive Data**: NO
- **Project Tags**: `[UNIVERSAL]` `[LIBRARY]`
- **Release Phase**: development

## Code Standards
- Maximum function/method length: 50 lines. If a function reads like it does two things, decompose it.
- Split a file when you find yourself using "and" to describe what it does — not when it hits a line count.
- Maximum function parameters: 5. If more, use a parameter object.
- No circular imports — module dependency graph must be acyclic (hook-enforced).
- `tsconfig.json` must include `"strict": true` AND `"noUncheckedIndexedAccess": true`.
  `strict: true` alone does not narrow `process.env.*` from `string | undefined` — the second flag is required
  to catch unguarded environment variable access at compile time.
- Every public function/method must have a JSDoc comment with typed params and returns.
- Delete orphaned code. Do not comment it out. Git has history.
- Before creating a new utility, search the entire codebase for existing ones.
- Reuse existing patterns — check shared modules before writing new.
- No abbreviations in names except universally understood ones (id, url, http, db, api).
- All names must be intention-revealing. If you need a comment to explain what a variable
  holds, the name is wrong.

## Dev Environment Hygiene

AI-assisted development can silently fill disk space. These rules are non-negotiable.
A full disk kills every running tool simultaneously — VS Code, Docker, the terminal, the DB.

### VS Code Extensions
- Before installing any extension: `code --list-extensions | grep -i <name>`.
- Only install if no version in the required major range is already present.
- Never run `code --install-extension` unconditionally in scripts or setup steps.
- Installing the same extension twice on the same day = a bug in your script.

### Docker Containers & Volumes
- Check before creating: `docker ps -a --filter name=<service>` — if it exists, start it, don't create it.
- Prefer `docker compose up` (reuse) over bare `docker run` (always creates new).
- One Compose file per project. Split files for the same project = tech debt.
- Log pruning: run `docker system prune -f` periodically. Never let container logs exceed 500 MB total.
- Time-series or synthetic data volumes: before writing >100 MB, ask whether raw retention,
  statistical condensation, or deletion after the run is preferred.
- Synthetic datasets older than 7 days with no code reference: ask to delete.

### Python Virtual Environments
- One `.venv` per project root, one per standalone package subdirectory — never more.
- Before creating: check if `.venv/` exists and `python --version` matches the required major.minor.
  Recreate only on major version mismatch or explicit user request.
- Never create a venv in a subdirectory unless that directory is a standalone installable package.
- Sanitize dependencies: if `pip list --not-required` reveals packages not in requirements, flag them.

### General Install Hygiene
- Before any install/download: check version already installed. Skip if within the required range.
- If project directory disk usage outside of `node_modules/`, `.venv/`, `dist/`, `.next/`
  exceeds 2 GB: surface a warning and ask before continuing any file-generating operation.
- Never silently grow the workspace. When uncertain about retention, ask.

## Dependency Registry — AI-Maintained Security Contract

The project's approved dependency set is a **living GS artifact maintained by the AI
assistant**. It is not a template rule — template authors cannot predict which library
will gain a CVE next quarter. The AI can run an audit at the moment a dependency is
about to be added. This block prescribes that it must.

### The registry artifact

File: **`docs/approved-packages.md`** — emit in P1 alongside schema, tsconfig, package.json.
Update it every time a dependency is added or upgraded. If it exists only in prose or a
README reference, it does not exist.

```markdown
# Approved Packages

| Package | Version range | Purpose | Alternatives rejected | Rationale | Audit status |
|---|---|---|---|---|---|
| example-pkg | ^2.4 | HTTP client | axios (larger bundle), node-fetch (no TS types) | Wide adoption, zero known CVEs | 0 HIGH/CRITICAL |
```

The AI populates every row. The registry is the authoritative record of WHY each
dependency was chosen and that it was clean at the time of addition.

### Process rules — stack-agnostic

1. **Before adding any package**: run the project's audit command (see table below)
   with `--dry-run` or equivalent to check the candidate for known CVEs.
   - If HIGH or CRITICAL found: choose an alternative and document the rejection.
   - If no CVE-free alternative exists: document the accepted risk and create an ADR
     naming the approver. Zero-tolerance is the default; exceptions require a record.
2. **After adding a package**: add a row to `docs/approved-packages.md` with audit status.
3. **Commit gate**: the pre-commit hook runs the audit command. HIGH or CRITICAL blocks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [K-Arthur/opencode-harness](https://github.com/K-Arthur/opencode-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
