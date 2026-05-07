---
trigger: always_on
description: - Provide expert-level review of the repository's structure, technology stack, and critical patterns, with a focus on Model Context Protocol (MCP) best practices
---

<Goals>
- Provide expert-level review of the repository's structure, technology stack, and critical patterns, with a focus on Model Context Protocol (MCP) best practices
- Highlight any non-obvious dependencies or architectural decisions that may impact code changes
- Ensure best practices are followed in terms of code organization, validation, and error handling
- Ensure architectural consistency across the codebase, especially in how MCP tools are implemented and how Census API interactions are handled
</Goals>

<HighLevelDetails>
This repository is a Model Context Protocol (MCP) server that provides AI-ready U.S. Census Bureau data.

**Technology Stack:**
- Language: TypeScript (Node.js runtime)
- API: U.S. Census Bureau API
- Database: PostgreSQL (for caching dataset metadata)
- Validation: Zod schemas
- Testing: Vitest
- Build: TypeScript compiler (tsc)
- Package Manager: npm
- Containerization: Docker & Docker Compose

**Repository Size:** Medium-sized TypeScript project with:
- MCP server application (mcp-server/)
- Database ETL service (mcp-db/)
- Multi-profile Docker Compose setup (dev/test/prod)
- Comprehensive test suite with unit and integration tests

**Project Purpose:** To enable use of official Census Bureau statistics with AI assistants, leveraging the Model Context Protocol to provide token-optimized data that reduces hallucinations.
</HighLevelDetails>

<BuildInstructions>
**Prerequisites:**
- Docker and Docker Compose installed
- Node.js 18+ (for local development outside Docker)

**Local Development Setup:**

The project uses Docker Compose with multiple profiles (dev, test, prod).

**Start Development Environment:**
```bash
docker compose --profile dev up
```
This starts:
- PostgreSQL database on port 5433
- Runs migrations automatically
- Leaves the dev container running for interactive commands

**Run Tests in Development:**
```bash
docker compose --profile dev exec census-mcp-db-dev-init npm run test
```

**Run Linter in Development:**
```bash
docker compose --profile dev exec census-mcp-db-dev-init npm run lint
```

**Run Tests (Standalone):**
```bash
docker compose --profile test up census-mcp-db-test-init
```
Starts test database and runs the test suite, then exits.

**Production Deployment:**
```bash
docker compose --profile prod up
```
Starts production services with MCP server on standard PostgreSQL port 5432.

**Validation:**
All validation (build, lint, tests) is handled by CI/GitHub Actions on pull requests.

**Important Notes:**
- The project is containerized - all services run in Docker
- Development uses port 5433, test uses 5434, prod uses 5432
- Database migrations run automatically on container startup
- The MCP server runs as a containerized service, not standalone
- Focus on writing correct code - let CI handle validation
- If modifying database schema, migrations will auto-run on next startup
</BuildInstructions>

<ProjectLayout>
**Directory Structure:**
```
/
├── mcp-db/                       # Database ETL service (CRITICAL - runs first)
│   ├── src/
│   │   ├── seeds                 # Orchestrates seeding process
│   │   │   ├── configs           # Configs for seeding different data types
│   │   │   ├── scripts           # Generic seeding utilities
│   │   │   │   ├── seed-runner   # Orchestrates seeding process
│   │   │   │   └── seed-database # Executes ETL pipeline
│   ├── migrations/               # Database schema migrations
│   ├── tests/                    # Test suite (mirrors src/)
│   ├── Dockerfile
│   ├── package.json
│   ├── tsconfig.json
│   ├── vitest.config.ts
│   └── eslint.config.js
├── mcp-server/                  # Main MCP server application
│   ├── src/
│   │   ├── tools/               # MCP tool implementations
│   │   │   ├── base.tool.ts
│   │   │   ├── fetch-aggregate-data.tool.ts
│   │   │   ├── fetch-dataset-geography.tool.ts
│   │   │   ├── list-datasets.tool.ts
│   │   │   └── resolve-geography-fips.tool.ts
│   │   ├── types/               # TypeScript definitions
│   │   ├── schema/              # Zod validation schemas
│   │   ├── prompts/             # MCP prompts
│   │   ├── services/            # Database service
│   │   ├── index.js             # Entry point
│   │   └── server.js
│   ├── tests/                   # Test suite (mirrors src/)
│   ├── Dockerfile
│   ├── package.json
│   ├── tsconfig.json
│   ├── vitest.config.ts
│   └── eslint.config.js
├── docker-compose.yml           # Multi-profile orchestration
```

**Configuration Files:**
- `mcp-server/tsconfig.json` - TypeScript strict mode, local development
- `mcp-server/tsconfig.docker.json` - TypeScript config for Docker builds
- `mcp-server/vitest.config.ts` - Test configuration with coverage
- `mcp-server/eslint.config.js` - Linting rules
- `mcp-server/.prettierrc` - Code formatting rules
- `mcp-db/Dockerfile` - Multi-stage build (dev/prod targets) for ETL service
- `mcp-db/tsconfig.json` - TypeScript config for database ETL
- `mcp-db/package.json` - Dependencies for migration and seeding
- `docker-compose.yml` - Multi-profile service orchestration (dev/test/prod)

**Key Architectural Elements:**

1. **MCP Database Service** (`mcp-db/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uscensusbureau/us-census-bureau-data-api-mcp](https://github.com/uscensusbureau/us-census-bureau-data-api-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
