---
trigger: always_on
description: This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.
---

# Agent Instructions

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

## Session Start (Zero-Context Agent)

If you spawn into this repo without context, do this first:

1. Read `docs/newcomer-guide.md` first for a practical repo orientation.
2. Open `docs/README.md` then `docs/INDEX.md` to get the current doc map.
3. Identify your task domain:
   - CLI behavior: `cli/cmd/ao/`, `cli/internal/`, `cli/docs/COMMANDS.md`
   - Skill behavior: `skills/<name>/SKILL.md`
   - Hook/gate behavior: `hooks/hooks.json` + `hooks/*.sh`
   - Validation/release/security flows: `scripts/*.sh` + `tests/`
4. Use source-of-truth precedence when docs disagree:
   1. Executable code and generated artifacts (`cli/**`, `hooks/**`, `scripts/**`, `cli/docs/COMMANDS.md`)
   2. Skill contracts and manifests (`skills/**/SKILL.md`, `hooks/hooks.json`, `schemas/**`)
   3. Explanatory docs (`docs/**`, `README.md`)
5. If you find conflicts, follow the higher-precedence source and call out the mismatch explicitly in your report/PR.

## Installing/Updating Skills

Use the [skills.sh](https://skills.sh/) npm package to install AgentOps skills for any agent:

```bash
# Claude Code: use Claude plugin install path (not npx)
claude plugin marketplace add boshu2/agentops
claude plugin install agentops@agentops-marketplace

# Codex CLI: installs the native plugin, archives stale raw mirrors when needed, then open a fresh Codex session
curl -fsSL https://raw.githubusercontent.com/boshu2/agentops/main/scripts/install-codex.sh | bash

# OpenCode
curl -fsSL https://raw.githubusercontent.com/boshu2/agentops/main/scripts/install-opencode.sh | bash

# Other agents (for example Cursor): install only selected skills
bash <(curl -fsSL https://raw.githubusercontent.com/boshu2/agentops/main/scripts/install.sh)
bash <(curl -fsSL https://raw.githubusercontent.com/boshu2/agentops/main/scripts/install.sh)

# Update all installed skills
bash <(curl -fsSL https://raw.githubusercontent.com/boshu2/agentops/main/scripts/install.sh)
```

## Quick Reference

```bash
# Issue tracking
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd vc status          # Inspect Dolt state if needed (JSONL auto-sync is automatic)

# CLI development
cd cli && make build  # Build ao binary
cd cli && make test   # Run tests
cd cli && make lint   # Run linter

# Validation (run before pushing)
scripts/pre-push-gate.sh --fast  # Smart conditional gate (only checks relevant to changed files)
bash scripts/install-dev-hooks.sh  # Activate repo-managed git hooks once per clone/worktree
scripts/ci-local-release.sh     # Full local release gate (runs everything)
scripts/validate-go-fast.sh     # Quick Go validation (build + vet + test)
```

## CI Validation — Passing the Pipeline

All pushes to `main` and PRs run `.github/workflows/validate.yml`. **Run checks locally before pushing.** The summary job gates on all checks except security-toolchain-gate (non-blocking), doctor-check (non-blocking), and check-test-staleness (non-blocking).
Blocking policy list (must match the validate summary failset): every job in the CI table below except jobs marked `(non-blocking)`, including `codex-runtime-sections`.

### Local Pre-Push Checklist

Run `scripts/pre-push-gate.sh --fast` for a smart conditional gate that only checks what changed. Or run individual checks below. If any fail, CI will fail too.

```bash
# Recommended: smart conditional gate
scripts/pre-push-gate.sh --fast

# Or individual checks:

# 1. Skill integrity (most common failure)
bash skills/heal-skill/scripts/heal.sh --strict

# 2. Doc-release gate (skill counts, link validation)
./tests/docs/validate-doc-release.sh

# 3. ShellCheck
find . -name "*.sh" -type f -not -path "./.git/*" -print0 | xargs -0 shellcheck --severity=error

# 4. Markdownlint
git ls-files '*.md' | xargs markdownlint

# 5. Go build + tests (if cli/ changed)
cd cli && make build && make test

# 6. Contract compatibility
./scripts/check-contract-compatibility.sh

# 7. Hook/docs parity
bash scripts/validate-hooks-doc-parity.sh

# 8. CI policy/docs parity
bash scripts/validate-ci-policy-parity.sh

# 9. Worktree disposition
bash scripts/check-worktree-disposition.sh

# 10. Plugin structure (symlinks, manifests)
./scripts/validate-manifests.sh --repo-root .
find skills -type l  # must be empty — zero symlinks allowed

 # 11. Headless runtime skill smoke (local Claude/Codex sessions; skips missing CLIs)
 bash scripts/validate-headless-runtime-skills.sh

 # 12. Codex-first override coverage (full skill catalog is classified and covered)
 bash scripts/validate-codex-override-coverage.sh

 # 13. Codex RPI contract and lifecycle guard checks
 bash scripts/validate-codex-rpi-contract.sh
 bash scripts/validate-codex-lifecycle-guards.sh

 # 14. Codex semantic parity audit (generated skills still match Codex-native tool/runtime semantics)
 bash scripts/audit-codex-parity.sh

# Full gate (runs everything above and more):
scripts/ci-local-release.sh
```

### CLI Skill-Map Refresh


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [boshu2/agentops](https://github.com/boshu2/agentops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
