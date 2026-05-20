---
trigger: always_on
description: Monorepo containing Nav's GitHub Copilot ecosystem tools:
---

# Copilot Instructions for navikt/copilot

## Repository Overview

Monorepo containing Nav's GitHub Copilot ecosystem tools:

- **my-copilot** - Self-service portal for managing Copilot subscriptions (Next.js 16)
- **copilot-metrics** - Naisjob that populates BigQuery with daily Copilot usage metrics (Go)
- **mcp-onboarding** - Reference MCP server with GitHub OAuth (Go)
- **mcp-registry** - Public registry for Nav-approved MCP servers (Go)

All applications deployed on NAIS platform with environment-specific configurations.

---

# Nav Development Standards

These standards apply across Nav projects. Project-specific guidelines follow below.

## Nav Principles

- **Team First**: Autonomous teams with circles of autonomy, supported by Architecture Advice Process
- **Product Development**: Continuous development and product-organized reuse over ad hoc approaches
- **Essential Complexity**: Focus on essential complexity, avoid accidental complexity
- **DORA Metrics**: Measure and improve team performance using DevOps Research and Assessment metrics

## Nav Tech Stack

- **Backend**: Kotlin with Ktor, PostgreSQL, Apache Kafka
- **Frontend**: Next.js 16+, React, TypeScript, Aksel Design System
- **Platform**: Nais (Kubernetes on Google Cloud Platform)
- **Auth**: Azure AD, TokenX, ID-porten, Maskinporten
- **Observability**: Prometheus, Grafana Loki, Tempo (OpenTelemetry)

## Nav Code Standards

### Minimal Editing

When fixing a bug or implementing a feature, change only what is necessary. Do not rename variables, restructure working code, or refactor beyond the task at hand. Keep diffs small and focused so they are easy to review.

### Kotlin/Ktor Patterns

- ApplicationBuilder pattern for bootstrapping
- Sealed classes for environment configuration (Dev/Prod/Local)
- Kotliquery with HikariCP for database access
- Rapids & Rivers pattern for Kafka event handling

### Next.js/Aksel Requirements

- **CRITICAL**: Always use Aksel spacing tokens, never Tailwind padding/margin
- Mobile-first with responsive props: `xs`, `sm`, `md`, `lg`, `xl`
- Norwegian number formatting with space separators

### Nais Deployment

- Manifests in `.nais/` directory
- Required endpoints: `/isalive`, `/isready`, `/metrics`
- OpenTelemetry auto-instrumentation for observability

### Writing Effective Agents

Based on [GitHub's analysis of 2,500+ repositories](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/), follow these patterns when creating or updating agents in `.github/agents/`:

**Structure (in order):**

1. **Frontmatter** - Name and description in YAML
2. **Persona** - One sentence: who you are and what you specialize in
3. **Commands** - Executable commands early, with flags and expected output
4. **Related Agents** - Table of agents to delegate to
5. **Core Content** - Code examples over explanations (show, don't tell)
6. **Boundaries** - Three-tier system at the end

**Six Core Areas to Cover:**

- Commands (with flags and options)
- Testing patterns
- Project structure
- Code style (✅ Good / ❌ Bad examples)
- Git workflow
- Boundaries

**Three-Tier Boundaries:**

```markdown
## Boundaries

### ✅ Always
- Run `mise check` after changes
- Use parameterized queries

### ⚠️ Ask First
- Modifying production configs
- Changing auth mechanisms

### 🚫 Never
- Commit secrets to git
- Skip input validation
```

**Key Principles:**

- **Commands early**: Put executable commands near the top, not buried at the bottom
- **Code over prose**: Show real code examples, not descriptions of what code should do
- **Specific stack**: Include versions (`Next.js 16`, `Go 1.26`, `Kotlin 2.0`)
- **Actionable boundaries**: "Never commit secrets" not "I cannot access secrets"

---

# Application-Specific Guidelines

## apps/my-copilot (Next.js + TypeScript)

Self-service portal for managing GitHub Copilot subscriptions. Next.js 16 app with Azure AD authentication.

### Commands

Working directory: `apps/my-copilot`

**Run after all changes:** `mise check`

**Available tasks:**

- `mise check` - Run all checks (ESLint, TypeScript, Prettier, Knip, Vitest)
- `mise test` - Run Vitest tests
- `mise dev` - Start Next.js dev server (http://localhost:3000)

### Tech Stack

- Next.js 16 with App Router
- TypeScript strict mode
- Nav Design System (@navikt/ds-react)
- Tailwind CSS v4.1
- Octokit for GitHub API
- BigQuery for usage analytics (via @google-cloud/bigquery)
- Vitest for testing

### File Structure

```
apps/my-copilot/src/
├── app/              # Next.js App Router pages
│   ├── api/         # API routes for Copilot data
│   ├── usage/       # Analytics dashboard
│   └── overview/    # License management
├── components/       # Reusable React components
└── lib/             # Utilities and business logic
    ├── auth.ts      # Azure AD JWT validation
    ├── github.ts    # GitHub API client (Octokit)
    ├── bigquery.ts  # BigQuery client for usage metrics
    ├── cached-bigquery.ts # Cached BigQuery queries
    └── data-utils.ts # Data aggregation
```

### Key Patterns

**Authentication:**

```typescript
const user = await getUser(); // redirects if not auth
const user = await getUser(false); // returns null if not auth
```

**API Routes:**

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [navikt/copilot](https://github.com/navikt/copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
