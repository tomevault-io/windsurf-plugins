---
trigger: always_on
description: This repo is a plugin containing agent skills (markdown-based knowledge packages) for Azure. Plugin source is under `plugin/`; the build produces versioned output in `output/`.
---

# GitHub Copilot for Azure — Repository Instructions

This repo is a plugin containing agent skills (markdown-based knowledge packages) for Azure. Plugin source is under `plugin/`; the build produces versioned output in `output/`.

## Repository Layout

```
plugin/                   # Plugin source (skills, hooks, MCP config, manifests)
  .plugin/plugin.json     # GitHub Copilot plugin manifest
  .cursor-plugin/         # Cursor plugin manifest
  .claude-plugin/         # Claude plugin manifest
  skills/<name>/          # Individual skill directories
    SKILL.md              # Skill definition (required)
    version.json          # NBGV per-skill version config
    references/           # On-demand reference docs
  hooks/                  # Agent hooks
  .mcp.json               # MCP server declarations
  version.json            # NBGV plugin-level version config

output/                   # Build output (git-ignored) — stamped, ready to deploy
scripts/                  # Dev tooling: token analysis, frontmatter/reference validators
evals/                    # Vally test suites
tests/                    # Jest test suite (unit, trigger, integration)
.github/
  instructions/           # Copilot instruction files for skill authoring
  skills/                 # Repo-local agent skills (not shipped in plugin)
  workflows/              # CI/CD workflows
docs/                     # Documentation (versioning, specs, diagrams)
eng/                      # Engineering scripts (test subscription cleanup)
gulpfile.ts               # Build pipeline
.token-limits.json        # Token budget config
.vally.yaml                # Vally eval framework config
```

## Building

```bash
npm install          # Install root + scripts deps (postinstall handles scripts/)
npm run build        # Copies plugin/ → output/, stamps NBGV versions, generates CHANGELOG.md
```

## Versioning Rules

This repo uses **Nerdbank.GitVersioning (NBGV)**. Versions are computed automatically from git commit history.

- **Never manually edit version numbers** in `plugin.json` or SKILL.md frontmatter under `plugin/`
- Source files must always use `"0.0.0-placeholder"` — the build stamps real versions
- Each skill has its own `version.json` with `pathFilters: ["."]`; only commits touching that skill's directory increment its version
- For skills outside `plugin/` (e.g., `.github/skills/`), set a real semver version and bump it in the same PR that modifies the skill
- Use conventional commit-style PR titles (e.g. `feat:`, `fix:`, `feature:`) — the build generates `CHANGELOG.md` from these

## Validating Changes

### Token and Structure Validators (from repo root)

```bash
npm run tokens check          # Check token limits against .token-limits.json
npm run tokens compare        # Compare token counts vs main
```

### Frontmatter and Reference Validation (from scripts/)

```bash
cd scripts
npm run frontmatter           # Validate skill YAML frontmatter against agentskills.io spec
npm run references            # Validate markdown links stay within skill directories
```

### Unit and Trigger Tests (from tests/)

```bash
cd tests
npm install
npm test                                    # Run all tests
npm test -- --testPathPatterns=<skill-name>  # Run tests for a single skill
npm run typecheck                            # TypeScript type checking
npm run lint                                 # ESLint
```

### Integration Tests

Integration tests require the Copilot SDK and run against a live agent:

```bash
cd tests
npm run test:integration -- <skill-name>
```

Skip integration tests when the SDK is unavailable:
```bash
SKIP_INTEGRATION_TESTS=true npm test -- --testPathPatterns=<skill-name>
```

## Adding a New Skill

> ⚠️ The char-count budget for skill descriptions is close to the Copilot CLI limit. Adding new skills risks truncation. Consider extending an existing skill first.

### Steps

1. **Create the skill directory**: `plugin/skills/<your-skill-name>/`

2. **Add `version.json`**:
   ```json
   {
     "version": "1.1",
     "pathFilters": ["."]
   }
   ```

3. **Write `SKILL.md`** with required frontmatter:
   ```yaml
   ---
   name: your-skill-name
   description: "What the skill does and when to use it. Include trigger phrases."
   license: MIT
   metadata:
     author: Microsoft
     version: "0.0.0-placeholder"
   ---
   ```
   - `name` must match the directory name (lowercase, hyphens only, 1-64 chars)
   - `version` must be `"0.0.0-placeholder"` — NBGV stamps the real version at build time
   - `description` must be 1-1024 chars, explaining WHAT and WHEN with trigger phrases

4. **Required sections** in SKILL.md: Quick Reference, When to Use This Skill, MCP Tools, Workflow/Steps, Error Handling

5. **Move detailed content** to `references/` subdirectory — keep SKILL.md under 500 tokens (soft limit)

6. **Add to `tests/skills.json`**: Add your skill name to the `skills` array and assign it to an integration test schedule slot

7. **Scaffold tests**: Copy `tests/_template` to `tests/<your-skill-name>/` and update `SKILL_NAME` in each test file

8. **Validate**:
   ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/GitHub-Copilot-for-Azure](https://github.com/microsoft/GitHub-Copilot-for-Azure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
