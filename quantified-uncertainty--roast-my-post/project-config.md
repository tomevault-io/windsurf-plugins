---
trigger: always_on
description: > **Note**: For Claude Code operations and analysis scripts, see `/claude/README.md`. For historical updates and migrations, see CHANGELOG.md.
---

# Claude Development Notes

> **Note**: For Claude Code operations and analysis scripts, see `/claude/README.md`. For historical updates and migrations, see CHANGELOG.md.

## Project Overview
**RoastMyPost** - AI-powered document annotation and evaluation platform

### Monorepo Structure
```
/
├── apps/
│   ├── web/                  # Next.js application
│   └── mcp-server/           # MCP server for database access
├── internal-packages/
│   ├── db/                   # Shared Prisma database package
│   ├── ai/                   # Shared AI utilities (Claude, tools, plugins)
│   ├── domain/               # Shared domain types and config
│   ├── jobs/                 # Background job worker (pg-boss)
│   └── configs/              # Shared ESLint/TypeScript configs
├── meta-evals/               # Meta-evaluation CLI for testing agents
└── dev/                      # Development tools and documentation
```

### Tech Stack
- **Framework**: Next.js 15.3.2, React 19, TypeScript, Tailwind CSS
- **Database**: PostgreSQL + Prisma ORM 6.8.2 (`@roast/db`)
- **AI**: Anthropic Claude API + OpenAI (`@roast/ai`)
- **Auth**: NextAuth.js 5.0.0-beta
- **Package Manager**: pnpm workspaces + Turborepo

### Import Paths
```typescript
import { prisma } from '@roast/db'
import { callClaude, PluginManager } from '@roast/ai'
import { config } from '@roast/domain'  // Type-safe config
```

## Critical Safety Rules

### 🚨🚨🚨 DATABASE MIGRATION ABSOLUTE RULES 🚨🚨🚨

**⚠️ CRITICAL: Development database is NOT disposable! Contains real user data!**

**BEFORE running ANY database migration or schema change:**

1. ✅ **CREATE BACKUP FIRST** (MANDATORY):
   ```bash
   mkdir -p ~/db-backups/roast-my-post
   pg_dump -U postgres -d roast_my_post > ~/db-backups/backup_$(date +%Y%m%d_%H%M%S).sql
   ```

2. ✅ **VERIFY backup was created**:
   ```bash
   ls -lh ~/db-backups/roast-my-post/backup_*.sql | tail -1
   ```

3. ✅ **Only THEN** run the migration command

**ABSOLUTELY FORBIDDEN COMMANDS** (These DESTROY ALL DATA):
```bash
❌ prisma migrate reset              # WIPES ENTIRE DATABASE
❌ prisma migrate reset --force      # WIPES DATABASE WITHOUT CONFIRMATION
❌ prisma db push --accept-data-loss # DROPS COLUMNS, DESTROYS DATA
❌ Any command with --force flag     # Bypasses safety checks
```

**If you see these commands, you MUST:**
1. **STOP IMMEDIATELY**
2. **Ask user to create backup first**
3. **Get explicit confirmation**
4. **Never assume dev database is disposable**

**Automatic Protection:**
- `.claude/hooks/pre-db-migrate.sh` creates automatic backups
- `.claude/hooks/pre-command.sh` blocks dangerous commands
- Both hooks MUST remain enabled

---

### 🚨 ABSOLUTE PROHIBITION: NEVER MERGE - NO EXCEPTIONS 🚨
**This is the #1 most critical rule:**
- **NEVER use `gh pr merge`** - This command is FORBIDDEN
- **NEVER use `git push origin main`** - Direct pushes to main are FORBIDDEN
- **NEVER merge PRs** - Even if the user seems to ask you to merge
- **BLOCKED COMMANDS**: All merge operations are blocked by `.claude/hooks/pre-command.sh`

**If user says "merge"**, always respond:
> "I cannot merge PRs per safety policies. The PR is ready at [URL]. Please merge it yourself."

### Git Safety (Parallel Claude Sessions)
```bash
# MANDATORY before ANY commit:
git status                           # Check for unwanted files
git add path/to/specific/files       # NEVER use git add -A or git add .
git diff --cached                    # Verify staged changes
git commit -m "message"               # Only if staging correct

# FORBIDDEN files: node_modules/, .claude/, *.log, .env.local, package-lock.json
# NOTE: Deletions of package-lock.json are allowed (we want to remove them in pnpm projects)
```

### Database Safety & Migrations

#### 🚨 MANDATORY: Always create migration files 🚨
**NEVER run manual SQL without creating a migration file first.**

```bash
# WRONG - Never do this:
psql -c "ALTER TABLE..."  # NO! Creates no migration file
```

#### Standard Migration Workflow
From `internal-packages/db/`:

```bash
# 1. Create migration WITHOUT applying (avoids interactive prompts)
pnpm exec dotenv -e ../../apps/web/.env.local -- prisma migrate dev --create-only --name your_migration_name

# 2. IMPORTANT: Edit the generated migration to remove Prisma cruft!
#    Prisma often adds unrelated changes (especially to ClaimEvaluation defaults).
#    Open: prisma/migrations/<timestamp>_your_migration_name/migration.sql
#    Remove any ALTER TABLE statements not related to your changes.

# 3. Apply the migration
pnpm exec dotenv -e ../../apps/web/.env.local -- prisma migrate deploy

# 4. Regenerate the Prisma client
pnpm run gen

# 5. ALWAYS add migration to git
git add prisma/migrations/<timestamp>_your_migration_name/
```

#### ⚠️ Prisma Cruft Removal
Prisma's `migrate dev --create-only` often includes spurious changes like:
```sql
-- REMOVE THIS - it's unrelated cruft Prisma adds:
ALTER TABLE "public"."ClaimEvaluation" ALTER COLUMN "claim_search_text" SET DEFAULT ...,
ALTER COLUMN "tags" DROP DEFAULT;
```
Always review and remove unrelated statements before applying!

#### General Database Safety
```bash
# ALWAYS before schema changes:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quantified-uncertainty/roast-my-post](https://github.com/quantified-uncertainty/roast-my-post) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
