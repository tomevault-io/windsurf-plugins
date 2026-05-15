---
trigger: always_on
description: - Build: `npm run build` - Compiles TypeScript to JavaScript
---

# Tiger Salesforce MCP Server - Development Guidelines

## Build, Test & Run Commands

- Build: `npm run build` - Compiles TypeScript to JavaScript
- Watch mode: `npm run watch` - Watches for changes and rebuilds automatically
- Run server: `npm run start` - Starts the MCP server using stdio transport
- Prepare release: `npm run prepare` - Builds the project for publishing

## Code Style Guidelines

- Use ES modules with `.js` extension in import paths
- Strictly type all functions and variables with TypeScript
- Follow zod schema patterns for tool input validation
- Prefer async/await over callbacks and Promise chains
- Place all imports at top of file, grouped by external then internal
- Use descriptive variable names that clearly indicate purpose
- Implement proper cleanup for timers and resources in server shutdown
- Follow camelCase for variables/functions, PascalCase for types/classes, UPPER_CASE for constants
- Handle errors with try/catch blocks and provide clear error messages
- Use consistent indentation (2 spaces) and trailing commas in multi-line objects

## Database Schema

All Salesforce data is stored in the `salesforce` schema. Note: `case` is a reserved word in PostgreSQL and must always be quoted as `salesforce."case"`.

### salesforce."case"

Primary support case entity.

| Column             | Type           | Notes                                |
| ------------------ | -------------- | ------------------------------------ |
| id                 | varchar(18)    | PK                                   |
| case_number        | varchar(90)    | Human-readable case number           |
| subject            | varchar(765)   | Case subject                         |
| description        | varchar(32000) | Case description                     |
| status             | varchar(765)   | Open, Closed, etc.                   |
| priority           | varchar(765)   | Low, Medium, High                    |
| origin             | varchar(765)   | Email, Web, Phone, etc.              |
| type               | varchar(765)   | Case type                            |
| reason             | varchar(765)   | Case reason                          |
| is_closed          | boolean        |                                      |
| is_escalated       | boolean        |                                      |
| closed_date        | timestamptz    |                                      |
| created_date       | timestamptz    |                                      |
| last_modified_date | timestamptz    |                                      |
| is_deleted         | boolean        | Soft delete flag                     |
| contact_id         | varchar(18)    | FK -> contact                        |
| account_id         | varchar(18)    | FK -> account                        |
| owner_id           | varchar(18)    | FK -> user                           |
| entitlement_id     | varchar(18)    | FK -> entitlement                    |
| opportunity_c      | varchar(18)    | FK -> opportunity (custom)           |
| cloud_project_id_c | varchar(765)   | Cloud project ID (custom)            |
| cloud_service_id_c | varchar(765)   | Comma-delimited service IDs (custom) |

### salesforce.account

| Column                     | Type         | Notes                    |
| -------------------------- | ------------ | ------------------------ |
| id                         | varchar(18)  | PK                       |
| name                       | varchar(765) | Company name             |
| type                       | varchar(765) | Account type             |
| industry                   | varchar(765) | Industry classification  |
| account_tier_c             | varchar(765) | Tier level (custom)      |
| account_stage_c            | varchar(765) | Lifecycle stage (custom) |
| account_health_c           | varchar(765) | Health score (custom)    |
| customer_start_date_c      | date         | Customer since (custom)  |
| customer_end_date_c        | date         | Churned date (custom)    |
| current_billable_mrr_c     | numeric      | Current MRR (custom)     |
| arr_as_of_last_month_c     | numeric      | ARR last month (custom)  |
| churn_risk_c               | boolean      | Churn risk flag (custom) |
| owner_id                   | varchar(18)  | FK -> user               |
| parent_id                  | varchar(18)  | FK -> account            |
| customer_success_manager_c | varchar(18)  | FK -> user (custom)      |
| is_deleted                 | boolean      | Soft delete flag         |

### salesforce.contact

| Column               | Type         | Notes                        |
| -------------------- | ------------ | ---------------------------- |
| id                   | varchar(18)  | PK                           |
| name                 | varchar(363) | Full name (computed)         |
| first_name           | varchar(120) |                              |
| last_name            | varchar(240) |                              |
| email                | varchar(240) |                              |
| phone                | varchar(120) |                              |
| title                | varchar(384) | Job title                    |
| account_id           | varchar(18)  | FK -> account                |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timescale/tiger-salesforce-mcp-server](https://github.com/timescale/tiger-salesforce-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
