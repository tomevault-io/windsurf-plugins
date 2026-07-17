---
trigger: always_on
description: Handles schema introspection for: `postgresql`, `mysql`/`mariadb`, `mongodb`. For API-integrated sources stored in internal PostgreSQL (`dra_xxx` schemas), the caller passes the `dra_xxx` schema name and the service queries internal PostgreSQL directly. No source-type branching is needed in `SchemaCollectorService` for new API-integrated sources — it works automatically as long as data exists in the `dra_new_type` schema.
---

# Data Research Analysis Platform - AI Coding Agent Instructions

## Project Overview
Full-stack data analytics platform (similar to Tableau/Power BI) built with Vue3/Nuxt3 SSR frontend, Node.js/Express/TypeScript backend, and PostgreSQL. Users connect multiple data sources (PostgreSQL, MySQL, MariaDB, CSV, Excel, PDF), build data models with AI assistance, and create interactive dashboards.

## Operational Restrictions
- **Manual Verification Required**: Do NOT use automated processes for checking code structure or verifying issues. Conduct all checks manually and carefully. If an automated process fails once, do not rely on it for subsequent verification; always verify manually.

## Planning Mode — "Plan Only" Requests

When the user says **"plan only"**, **"planning only"**, or any equivalent phrasing, do NOT write or modify any code. Instead:

1. **Brainstorm first.** Before producing any plan, ask the user targeted clarifying questions using the `ask_questions` tool. The goal is to:
   - Resolve ambiguities that would force assumptions in the plan
   - Identify hidden complexity early so the design stays simple
   - Confirm scope boundaries (what is in vs out of the PR)
   - Surface contradictions in the stated requirements and resolve them with the user

2. **Keep questions focused.** Ask a maximum of 4 questions per round, each with 2–5 concrete options where possible. Prefer questions whose answers materially change the architecture or implementation approach.

3. **Iterate if needed.** After the first round of answers, identify any remaining open questions or tensions in the responses and ask a follow-up round before finalising the plan. Do not proceed to writing the plan until the design is unambiguous.

4. **Then produce the plan.** Once questions are resolved, write a detailed, structured implementation plan that is:
   - Broken into numbered issues with clear size, priority, and dependencies
   - Specific about which files change and what the code patterns look like
   - Simple enough that a developer can implement each issue independently
   - Saved to a markdown file in `documentation/` if the user requests it

5. **Never make assumptions silently.** If a question has meaningful architectural consequences, always ask rather than guess.

---

## Architecture Essentials

### Backend: Singleton Processor Pattern
**Critical**: Business logic lives in singleton Processors (11 total), NOT controllers/routes. Controllers are thin orchestrators.

**Pattern Example** ([AuthProcessor.ts](backend/src/processors/AuthProcessor.ts)):
```typescript
export class AuthProcessor {
    private static instance: AuthProcessor;
    public static getInstance(): AuthProcessor {
        if (!AuthProcessor.instance) {
            AuthProcessor.instance = new AuthProcessor();
        }
        return AuthProcessor.instance;
    }
    // All auth business logic here
}
```

**Processors**: `AuthProcessor`, `ProjectProcessor`, `DataSourceProcessor`, `DataModelProcessor`, `DashboardProcessor`, `UserProcessor`, `UserManagementProcessor`, `TokenProcessor`, `ArticleProcessor`, `CategoryProcessor`, `PrivateBetaUserProcessor`

### Frontend: Pinia Stores with localStorage Sync
**Pattern**: All 9 Pinia stores automatically sync to localStorage on client ([projects.ts](frontend/stores/projects.ts)):
```typescript
function setProjects(projectsList: IProject[]) {
    projects.value = projectsList
    if (import.meta.client) {
        localStorage.setItem('projects', JSON.stringify(projectsList));
        enableRefreshDataFlag('setProjects');
    }
}
```

**Stores**: `projects`, `data_sources`, `data_models`, `dashboards`, `logged_in_user`, `articles`, `private_beta_users`, `user_management`, `ai-data-modeler`

### TypeORM Models with Automatic Encryption
**Critical**: Sensitive fields (connection_details, credentials) auto-encrypt via `ValueTransformer` ([DRADataSource.ts](backend/src/models/DRADataSource.ts)):
```typescript
const connectionDetailsTransformer: ValueTransformer = {
    to(value): any { return EncryptionService.getInstance().encrypt(value); },
    from(value): any { return EncryptionService.getInstance().decrypt(value); }
};

@Entity('dra_data_sources')
export class DRADataSource {
    @Column({ type: 'jsonb', transformer: connectionDetailsTransformer })
    connection_details!: IConnectionDetails
}
```

## Developer Workflows

### Setup & Migration Commands
```bash
# Create required volumes first
docker volume create data_research_analysis_postgres_data
docker volume create data_research_analysis_redis_data

# Build and start services
docker-compose build && docker-compose up

# Inside backend container/directory:
npm run migration:run                      # Run migrations
npm run seed:run -- -d ./src/datasources/PostgresDSMigrations.ts src/seeders/*.ts
npm run migration:generate -- --name=CreateNewFeature

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danieljoshua01/Data-Research-Analysis-Platform](https://github.com/danieljoshua01/Data-Research-Analysis-Platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
