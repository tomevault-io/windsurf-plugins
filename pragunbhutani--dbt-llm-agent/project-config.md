---
trigger: always_on
description: We are building an LLM powered AI data analyst for Data Engineering teams that work with dbt to manage their analytics code bases. To use this project, users should be able to connect with their dbt cloud projects or their dbt core github repos via an interface which is then used to build a knowlege base. We then use various AI based workflows to allow users to ask questions about their data and then surface the queries, charts or insights needed to answer their data questions.
---


# Project Description

We are building an LLM powered AI data analyst for Data Engineering teams that work with dbt to manage their analytics code bases. To use this project, users should be able to connect with their dbt cloud projects or their dbt core github repos via an interface which is then used to build a knowlege base. We then use various AI based workflows to allow users to ask questions about their data and then surface the queries, charts or insights needed to answer their data questions.

We will enable interaction through a variety of interfaces including Slack, MCP connectivity with LLM apps, a streaming chat interface and maybe even more in future.

We will also allow users to connect our app with their data warehouses so we can try and answer their questions directly, and support our answers with charts and visualisations etc. If not, we should surface the queries users can use to answer their questions.

We may add more functionality in future or get rid of some of the functionality I've mentioned. I've shared this information for context but we may not be building all of these things at once.

Ask clarifying questions and give your feedback on design decisions objectively - there is no need to agree with every design decision I propose, feel free to provide constructive feedback.

# Monorepo Structure

The project is a monorepo that looks something like this:

/ragstar-project-root/
├── backend_django/ # Django application (moved from root)
│ ├── manage.py
│ ├── ragstar/ # Django settings directory
│ ├── apps/ # Django apps
│ │ ├── accounts # user accounts, organisations, org settings etc.
│ │ ├── data_sources # connection to knowledge sources like dbt
│ │ ├── embeddings # storing and retrieving embeddings
│ │ ├── integrations # integrations to external tools like slack, metabase etc.
│ │ ├── knowledge_base # information about our dbt projects, models, questions etc.
│ │ ├── llm_providers # interface for interacting with LLM provider APIs
│ │ ├── workflows # all workflows, agentic or not
│ │ │ ├── workflow_name # each workflow has a workflow.py, prompts.py etc.
│ │ └── ... # More apps may be found here
│ ├── static/
│ ├── pyproject.toml # Python dependencies
│ ├── uv.lock # Lock file
│ ├── .python-version # Python version specification
│ └── Dockerfile # Backend-specific Docker config
├── frontend_nextjs/ # Renamed from client/ - ready for Next.js
│ ├── public/ # Public static assets
│ ├── src/
│ │ ├── app/ # Next JS App router project structure
│ │ │ ├── (auth)/ # Signin and Signup pages
│ │ │ ├── dashboard/ # Dashboard pages
│ │ │ ├── ...
│ │ ├── components/ # Creates reusable react components
│ │ └── ... # Other reusable utilities should be placed here
│ └── ... # NextJS, Typescript, package.json, eslint etc.
├── mcp_server # FastMCP server (FastMCP is starlette not FastAPI)
├── config_examples/ # Config examples like .slack_manifest.example.json, .ragstarrules.example.yml
├── docs/ # GitHub Pages docs (unchanged)
├── docker-compose.yml # Orchestrates all services
├── .env.example # Example environment file
└── .env # Shared environment variables

## Monorepo Projects

- backend_django
  - The API for our NextJS frontend, MCP server, Authentication, Third party integrations etc.
- frontend_nextjs
  - Admin panel used to configure our app, build and edit the knowledge base, study conversations etc.

### Monorepo Project Rules

#### frontend_nextjs

- We're using the app router with NextJS.
- Use auth.js for authentication.
- Use TailwindCSS classes for style only. Do not define custom or inline css.
- Examine the structure of the src/ folder before creating new directories to avoid duplication.
- The `shadcn-ui` package has been deprecated in favour of `shadcn`

#### backend_django

- All of our workflows, whether agentic or not are stored under apps/workflows.
- Each workflow has a workflow.py, prompts.py and other files that store this infomation consistently.

#### mcp_server

- Basic MCP server built using FastMCP.
  - FastMCP is not FastAPI and FastAPI middleware does not work with FastMCP

## Other Services Used

- postgres db with pgvector
- redis

## Authentication

This project uses next-auth/react on the frontend and Django REST Framework with Simple JWT on the backend. Here's the correct way to handle authenticated API requests from the Next.js client to the Django API:

1. Session Management: Authentication is handled by next-auth. The JWT access token required by the Django backend is made available on the client-side session object.
2. Fetching the Token: In any component that needs to make an authenticated API call, we must use the useSession hook from next-auth/react to get the current session. The access token is located at session.accessToken.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pragunbhutani/dbt-llm-agent](https://github.com/pragunbhutani/dbt-llm-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
