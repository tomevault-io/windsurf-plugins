---
trigger: always_on
description: Detailed guides extracted via progressive disclosure. CLAUDE.md contains essentials; sub-docs contain deep dives.
---

# Claude Code Configuration - Skillsmith

## Sub-Documentation

Detailed guides extracted via progressive disclosure. CLAUDE.md contains essentials; sub-docs contain deep dives.

| Document | Description |
|----------|-------------|
| [docker-guide.md](.claude/development/docker-guide.md) | Container rebuild scenarios, DNS failure, native modules, troubleshooting |
| [git-crypt-guide.md](.claude/development/git-crypt-guide.md) | Unlock, worktree setup, rebase workaround, smudge filter fixes |
| [ci-reference.md](.claude/development/ci-reference.md) | Branch protection, change classification, Turborepo, CI scripts |
| [deployment-guide.md](.claude/development/deployment-guide.md) | Edge function deploy commands, CORS, website, monitoring & alerts |
| [claude-flow-guide.md](.claude/development/claude-flow-guide.md) | Ruflo (formerly claude-flow) — agent types, swarm examples, hive mind, SPARC modes |
| [cloudinary-guide.md](.claude/development/cloudinary-guide.md) | Blog image upload workflow, URL transforms, folder conventions |
| [vscode-publishing-guide.md](.claude/development/vscode-publishing-guide.md) | VS Code Marketplace publishing, local/CI workflow, PAT rotation |
| [subagent-tool-permissions-guide.md](.claude/development/subagent-tool-permissions-guide.md) | Subagent tool access by type, foreground/background behavior, skill author checklist |
| [supabase-migration-safety.md](.claude/development/supabase-migration-safety.md) | Pre/post-apply query catalog, ACCESS EXCLUSIVE lock discipline, rollback convention, pooler rules. Invoke via `supabase-migration-reviewer` skill for automated review. |
| [ruvector-dev-tooling.md](.claude/development/ruvector-dev-tooling.md) | `skillsmith-doc-retrieval` MCP — local semantic doc search (SMI-4417). Setup, tool surface, privacy boundary, post-commit hook, token-delta gate. |

**Implementation plan template**: [.claude/templates/implementation-plan.md](.claude/templates/implementation-plan.md) — use this structure for all plans in `docs/internal/implementation/`.

---

## Docker-First Development

**All code execution MUST happen in Docker.** Native modules require glibc. See [ADR-002](docs/internal/adr/002-docker-glibc-requirement.md).

```bash
docker compose --profile dev up -d                    # Start container (REQUIRED first)
docker exec skillsmith-dev-1 npm run build             # Build
docker exec skillsmith-dev-1 npm test                  # Test
docker exec skillsmith-dev-1 npm run lint              # Lint
docker exec skillsmith-dev-1 npm run typecheck         # Typecheck
docker exec skillsmith-dev-1 npm run audit:standards   # Standards audit
docker exec skillsmith-dev-1 npm run preflight         # All checks before push
```

**After pulling changes**: The `post-merge` hook auto-runs `npm install` in Docker when `package-lock.json` changes. If the container is not running, start it and run `docker exec skillsmith-dev-1 npm install && docker exec skillsmith-dev-1 npm run build`.

**Full rebuild** (native module issues, major upgrades): See [docker-guide.md](.claude/development/docker-guide.md#full-rebuild-thorough).

**Container management**: `docker compose --profile dev down` (stop), `docker logs skillsmith-dev-1` (logs).

**Submodule**: Run `git submodule update --init` before `docker compose up` if internal docs are needed inside the container. Internal docs are not available inside Docker by default.

---

## CI Health Requirements

| Category | Requirement |
|----------|-------------|
| ESLint | Zero warnings, zero errors |
| TypeScript | Strict mode, no unjustified `any` |
| Prettier | All files formatted |
| Tests | 100% pass rate |
| Security | No high-severity vulnerabilities |
| File size | < 500 lines per file |
| Test coverage | > 80% |
| Test co-update | Source file changes must include related test updates |

**New source files must be under 500 lines.** Split into companion files (e.g., `foo.helpers.ts`, `foo.types.ts`) if approaching the limit. The `audit:standards` script enforces this.

**When CI fails**: Don't merge. Check logs. Run `docker exec skillsmith-dev-1 npm run preflight` locally. Create Linear issue if non-trivial.

**npm overrides** (transitive vulnerability fixes in root `package.json`):

- **Before adding an override**, check if the target is exact-pinned: `docker exec skillsmith-dev-1 node -e "console.log(require('<pkg>/package.json').dependencies['<dep>'])"`. If no `^`/`~` prefix, a flat override alone **will not work** — npm cannot replace exact-pinned versions. However, `npm update <pkg>` may resolve it via dedup if another chain pulls in the patched version. Verify with `npm ls <dep>` after update. If the exact-pinned copy persists, dismiss the alert with documented rationale.
- `ajv`: scoped overrides only (`"parent": { "ajv": "^8.18.0" }`). A global override breaks ESLint (`ajv@6.x` → `8.x` API incompatible).
- `typescript-eslint` is a meta-package — always update `typescript-eslint`, `@typescript-eslint/parser`, and `@typescript-eslint/eslint-plugin` together (they share internal version locks). Dependabot groups them automatically (see `.github/dependabot.yml`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smith-horn/skillsmith](https://github.com/smith-horn/skillsmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
