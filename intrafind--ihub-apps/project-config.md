---
trigger: always_on
description: This is a full-stack AI application platform built with **Node.js (Express)** and **React**, enabling companies to deploy and customize AI-powered applications without coding. The platform supports multiple LLM providers (OpenAI, Anthropic, Google, Mistral), flexible authentication (Anonymous, Local, OIDC, Proxy), and enterprise-grade features.
---

# GitHub Copilot Instructions for iHub Apps

This is a full-stack AI application platform built with **Node.js (Express)** and **React**, enabling companies to deploy and customize AI-powered applications without coding. The platform supports multiple LLM providers (OpenAI, Anthropic, Google, Mistral), flexible authentication (Anonymous, Local, OIDC, Proxy), and enterprise-grade features.

## Project Structure

- **`client/`**: React application (Vite + Tailwind CSS) with hot reload
- **`server/`**: Node.js Express backend with LLM adapters and authentication
- **`shared/`**: Code shared between client and server (utilities, i18n)
- **`contents/`**: JSON configuration files for apps, models, UI, groups, sources, tools
- **`examples/`**: Example and customer-specific configuration templates
- **`concepts/`**: Feature concept documents, design docs, RFCs (format: `YYYY-MM-DD {title}.md`)
- **`docs/`**: User-facing feature documentation (rendered to `/help` in production)
- **`tests/`**: Test files for server components and integrations

## Quick Setup

For new development environments, run:

```bash
# Quick setup (copies .env, installs all dependencies)
npm run setup:dev

# Edit .env with your API keys (OPENAI_API_KEY, ANTHROPIC_API_KEY, etc.)

# Start development environment (server on :3000, client on :5173)
npm run dev
```

### Additional Setup Notes

- Chrome/Chromium must be available in `PATH` for Playwright tools: `npx playwright install`
- API keys are loaded from `.env` file (never commit API keys to the repository)
- Server runs on port 3000, Vite dev server on port 5173
- Access frontend at `http://localhost:5173` during development

## Concepts and Documentation

### Feature Documentation (docs/)

**All user-facing feature documentation should be added to the `docs/` folder:**

**When to Update Existing Documentation:**
- **Always check first** if documentation already exists in `docs/` for the area you're working on
- Update existing files rather than creating new ones when the feature fits within an existing document
- For example:
  - New model features → add to `docs/models.md`
  - New UI features → add to `docs/ui.md`
  - New authentication features → add to `docs/authentication-architecture.md`
  - New configuration → add to relevant config docs

**When to Create New Documentation:**
- Only create new documentation files when the feature doesn't fit into any existing document
- Use descriptive, lowercase filenames with hyphens: `feature-name.md`
- Add the new file to `docs/SUMMARY.md` for inclusion in the documentation site

**Documentation Structure:**
- `docs/` - User-facing feature documentation, guides, and references
  - Updated as features are added or modified
  - Organized by topic (models, authentication, configuration, etc.)
  - Rendered on the documentation site at `/help`

### Concept Documents (concepts/)

Every new feature, bug fix, or significant change should have a concept document in the `concepts/` folder for design and planning purposes. Always check the concept regarding information. When implementing new features, make sure that a concept document exists. If none exists, always make sure to create one.
If one exists, make sure that you update it with decisions we have taken and where code related to the feature can be found.

**Always store the following in the concepts folder `concepts/` and format them `YYYY-MM-DD {title}.md`:**
- Feature concepts and design documents
- Fix summaries and root cause analyses
- Migration guides for breaking changes or major updates
- Implementation summaries

**For larger features with multiple documents, organize them in a dedicated subfolder:**
- Create a subfolder in `concepts/` with a descriptive name (e.g., `concepts/websearch-provider-api-keys/`)
- Place all related documents in that subfolder
- Include a `README.md` in the subfolder that provides an overview and links to the documents
- This keeps related documentation together and makes it easier to find

**Example naming:**
- Single document: `concepts/2026-02-02 Provider API Key Persistence Fix.md`
- Organized feature: 
  - `concepts/websearch-provider-api-keys/README.md`
  - `concepts/websearch-provider-api-keys/2026-02-03 Websearch Provider API Key Configuration.md`
  - `concepts/websearch-provider-api-keys/2026-02-03 Websearch Provider UI Screenshots.md`
  - `concepts/websearch-provider-api-keys/IMPLEMENTATION_SUMMARY_WEBSEARCH_PROVIDERS.md`

## Development Workflow

### Building and Running

```bash
# Development mode with hot reload
npm run dev

# Production build
npm run prod:build

# Start production server
npm run start:prod

# Check server health
npm run health

# View server logs
npm run logs
```

### Code Quality - CRITICAL ⚠️

**ALWAYS run linting and formatting before committing:**

```bash
# Auto-fix linting issues (REQUIRED before commits)
npm run lint:fix

# Auto-format all files (REQUIRED before commits)
npm run format:fix

# Combined command
npm run lint-format:fix
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intrafind/ihub-apps](https://github.com/intrafind/ihub-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
