---
trigger: always_on
description: **Status:** Experimental - Phase 2 of 4
---

# Rails AI Agent System

**Status:** Experimental - Phase 2 of 4
**Architecture:** Single agent + 12 domain skills

This document is internal documentation for contributors. Users get the agent files from `agents/` directory.

## Architecture

Rails-AI uses a **single-agent architecture** where the architect loads:
- **Superpowers workflows** - Process layer (brainstorming, TDD, debugging, code review)
- **Rails-AI skills** - Domain expertise (12 Rails-specific skills)
- **Team rules** - 20 conventions from TEAM_RULES.md

### Structure

```text
rails-ai/
├── agents/
│   └── architect.md           # Single coordinator agent
├── commands/
│   └── architect.md           # /rails-ai:architect command
├── skills/                    # 12 domain skills
│   ├── project-setup/
│   ├── controllers/
│   ├── debugging/
│   ├── hotwire/
│   ├── jobs/
│   ├── mailers/
│   ├── models/
│   ├── security/
│   ├── styling/
│   ├── testing/
│   ├── using-rails-ai/
│   └── views/
├── rules/
│   └── TEAM_RULES.md          # 20 team conventions
└── test/
    └── unit/                  # Fast unit tests only
```

## Skills

**12 domain-organized skills** with YAML front matter:

1. **project-setup** - Project validation, environment config, credentials, Docker, RuboCop (coordinates with other skills)
2. **controllers** - RESTful actions, strong parameters, concerns
3. **debugging** - Rails debugging tools + superpowers:systematic-debugging
4. **hotwire** - Turbo Drive, Frames, Streams, Morph, Stimulus
5. **jobs** - SolidQueue, SolidCache, SolidCable (NO Redis/Sidekiq)
6. **mailers** - ActionMailer with async delivery
7. **models** - ActiveRecord patterns, validations, associations
8. **security** - XSS, SQL injection, CSRF, file uploads
9. **styling** - Tailwind CSS, DaisyUI
10. **testing** - TDD with Minitest, fixtures, mocking
11. **using-rails-ai** - Introduction and architecture guide
12. **views** - Partials, helpers, forms, accessibility (WCAG 2.1 AA)

Each skill includes:
- YAML front matter (name, description)
- When to use
- Patterns and examples
- Anti-patterns to avoid
- Related skills/workflows

## Team Rules

**20 conventions** in `rules/TEAM_RULES.md`:

**Critical rules (REJECT violations):**
1. ❌ NEVER Sidekiq/Redis → ✅ SolidQueue/SolidCache
2. ❌ NEVER RSpec → ✅ Minitest only
3. ❌ NEVER custom routes → ✅ RESTful resources
4. ❌ NEVER skip TDD → ✅ RED-GREEN-REFACTOR
5. ❌ NEVER merge without review → ✅ Draft PRs
6. ❌ NEVER WebMock bypass → ✅ Mock all HTTP

See TEAM_RULES.md for all 20 rules with enforcement levels.

## Development

### Testing

```bash
rake test:unit              # All unit tests
rake test:unit:skills       # Skills only
rake test:unit:agents       # Agents only
rake test:unit:rules        # Rules only
bin/ci                      # Full check (lint + tests)
```

**Unit tests validate:**
- Agent structure and YAML front matter
- Skill structure and metadata
- Rules consistency and mappings
- No integration tests (removed)

### Adding Skills

1. Create `skills/domain/SKILL.md` with YAML front matter
2. Add unit tests in `test/unit/skills/domain_test.rb`
3. Document in `skills/using-rails-ai/SKILL.md` (skill-to-task mapping table)
4. **Update `skills/project-setup/SKILL.md`** if the new skill affects project verification:
   - Add to Step 1 if required for verification
   - Reference in Step 2 (Gemfile), Step 4 (config files), or other steps as needed
   - Update `<related-skills>` section
5. Update architect command if needed (`commands/architect.md`)
6. Run `bin/ci`

### Adding Rules

1. Add to `rules/TEAM_RULES.md`
2. Update quick lookup index
3. Set enforcement severity
4. Add tests in `test/unit/rules/`
5. **Update `skills/project-setup/SKILL.md`** if the rule affects project setup verification:
   - Add rule checks to Step 2 (Gemfile violations)
   - Add rule checks to Step 3 (project structure)
   - Add rule checks to Step 4 (configuration files)
   - Reference the appropriate domain skill for verification
6. Update domain skills that enforce the rule
7. Run `bin/ci`

### Modifying Architect

1. Edit `commands/architect.md`
2. Maintain YAML front matter structure
3. Reference Superpowers workflows correctly
4. Run `bin/ci`

### Updating Domain Skills

1. Edit `skills/domain/SKILL.md`
2. If adding gem requirements → Update `skills/project-setup/SKILL.md` Step 2
3. If adding configuration patterns → Update `skills/project-setup/SKILL.md` Step 4
4. If skill becomes required for verification → Add to `skills/project-setup/SKILL.md` Step 1
5. Update tests in `test/unit/skills/domain_test.rb`
6. Run `bin/ci`

### Maintaining Cross-Skill Consistency

**The project-setup skill coordinates verification** - when you update:

- **TEAM_RULES.md** → Ensure project-setup checks for violations
- **Domain skills** (jobs, testing, security, styling) → Ensure project-setup references them
- **Gem requirements** → Add to domain skill, ensure project-setup references it
- **Configuration patterns** → Add to domain skill, ensure project-setup references it

**Think of it as:**
- Domain skills = Authoritative source of truth
- project-setup = Orchestrator that references domain skills
- When domain changes, project-setup references should update

## Quality Checks

```bash
rake lint               # All linters

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zerobearing2/rails-ai](https://github.com/zerobearing2/rails-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
