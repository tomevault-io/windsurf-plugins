---
trigger: always_on
description: > **IMPORTANT: Never develop on `main` branch!**
---

# HR skills development guide

## Branch strategy

> **IMPORTANT: Never develop on `main` branch!**
>
> - `main` is the **publishing branch** — it only contains released skills
> - `dev` is the **development branch** — all work happens here

| Branch | Purpose | Direct commits |
|--------|---------|----------------|
| `main` | Publishing (`npx skills add tuanductran/hr-skills`) | Forbidden |
| `dev` | Development, tests, experiments | Via PR only |

## Commit convention

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```text
<type>(<scope>): <short summary>
```

### Types

| Type | When to use |
|------|-------------|
| `feat` | Add a new skill or a new feature to the tooling |
| `fix` | Fix a bug in a skill's content or in the tooling |
| `chore` | Maintenance tasks — deps, config, CI, `.gitignore` |
| `docs` | Changes to `docs/`, `README.md`, `AGENTS.md`, or any other documentation |
| `refactor` | Code restructuring with no behaviour change |
| `style` | Formatting, whitespace, markdownlint fixes |
| `test` | Add or update tests |
| `build` | Changes to the build system (`packages/`, `tsconfig`, `bun.lock`) |
| `ci` | Changes to GitHub Actions workflows |

### Scopes (optional)

Use the skill name or package name as scope when the change is isolated:

```text
feat(hr-recruiting): add ATS integration prompts
fix(hr-compliance): correct FMLA section length
chore(skills-ref): update bun dependency
docs(installation): add claude.ai paste method
```

### Rules

- Use the **imperative mood** in the summary: "add", "fix", "update" — not "added" or "fixes"
- Keep the summary under 72 characters
- Reference issues in the body when relevant: `Closes #42`
- Breaking changes: append `!` after the type and explain in the body

### Examples

```text
feat(hr-onboarding): add virtual onboarding prompts
fix(hr-analytics): correct turnover formula in tips
chore: upgrade markdownlint-cli to 0.47
docs: add skill-format specification to docs/
build(skills-ref): switch build target to bun
ci: add ESLint job to hr-skills-ci workflow
```

After completing any task, validate skills and run the linter:

```bash
bun install          # Install all dependencies (run once, or after package changes)
bun run sync         # Sync skill references across all docs (run after adding/removing a skill)
bun run validate     # Validate all 15 skill SKILL.md files
bun run lint         # ESLint on TypeScript sources (0 errors required)
bun run lint:fix     # ESLint with auto-fix
bun run lint:md      # markdownlint + case-police on Markdown files
bun run lint:md:fix  # Markdown lint with auto-fix
bun run typecheck    # Type-check all packages with tsc
bun run build        # Build packages (skills-ref CLI → dist/)
bun run catalog      # Regenerate skills/CATALOG.md
bun run zip          # Regenerate all skills/*.zip packages (requires the system `zip` CLI — install it first if not available)
```

When you add a new skill directory (for example `skills/hr-new-skill/SKILL.md`), run `bun run sync` first. It auto-updates `config.ts`, all documentation tables, `marketplace.json`, and skill counts across the project — no manual edits needed. Then run `bun run zip` to regenerate the zip packages.

## Packages

| Package | Description |
|---------|-------------|
| `packages/hr-skills-build` | `validate.ts` and `catalog.ts` scripts — run via `bun run validate` / `bun run catalog` from root |
| `packages/skills-ref` | TypeScript library + CLI (`skills-ref`) for reading, validating, and generating prompts from skill files |

## Skill scopes

| Skill | Scope |
|-------|-------|
| **hr-analytics** | Help HR managers with HR analytics and data management. |
| **hr-compensation-benefits** | Help HR managers with compensation and benefits programs. |
| **hr-compliance** | Help HR managers with HR compliance and workplace policies. |
| **hr-conflict-resolution** | Help HR managers with workplace conflict resolution. |
| **hr-diversity-inclusion** | Help HR managers with diversity, equity, and inclusion initiatives. |
| **hr-employee-engagement** | Help HR managers with employee engagement strategies. |
| **hr-employee-relations** | Help HR managers with employee relations matters. |
| **hr-leadership-development** | Help HR managers with leadership development programs. |
| **hr-onboarding** | Help HR managers with employee onboarding and offboarding. |
| **hr-performance-management** | Help HR managers with performance management processes. |
| **hr-recruiting** | Help HR managers with end-to-end recruiting and talent acquisition. |
| **hr-technology** | Help HR managers with HR technology strategy and implementation. |
| **hr-training-development** | Help HR managers with learning and development programs. |
| **hr-vietnam-context** | Vietnam-specific HR guidance covering the Labor Code, Social Insurance Law, personal income tax registration, work permits for foreign workers, trade union obligations, and cultural norms for managing teams in Vietnam. |
| **hr-workforce-planning** | Help HR managers with workforce planning and strategy. |

## Content standards

All documentation and skill content follows [Atlassian's content design guidelines](https://atlassian.design/foundations/content/). The key rules that apply to this project:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tuanductran/hr-skills](https://github.com/tuanductran/hr-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
