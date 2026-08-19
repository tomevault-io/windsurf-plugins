---
trigger: always_on
description: Polyglot monorepo with 26 independent QA/SDET frameworks across 6 languages (TypeScript, Java, C#, Python, JavaScript, HCL), plus 3 companion repos. Each sub-project is self-contained with its own dependencies, config, tests, and CI workflow. The repo targets SauceDemo, a Shopify test store, and custom FastAPI services as systems under test.
---

# CLAUDE.md — QA Automation Portfolio

## Repository overview

Polyglot monorepo with 26 independent QA/SDET frameworks across 6 languages (TypeScript, Java, C#, Python, JavaScript, HCL), plus 3 companion repos. Each sub-project is self-contained with its own dependencies, config, tests, and CI workflow. The repo targets SauceDemo, a Shopify test store, and custom FastAPI services as systems under test.

## Branching and commits

- **Never push directly to `main`.** Create a feature or fix branch (`dev`, `feature/*`, `fix/*`) and merge via PR.
- Write concise, conventional commit messages: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `ci:`.
- Never commit `.env` files, credentials, API keys, or secrets. Each framework provides a `.env.example` template — reference that pattern when adding new config.

## Monorepo structure

Each directory is an independent framework. Changes to one should not break another.

```
# Browser / E2E
cypress/               TypeScript — Cypress 15, React 18, Vite
playwright/            C# + TypeScript — Playwright 1.52, .NET 8, NUnit
selenium-java/         Java — Selenium 4, TestNG 7.10, Maven, Appium

# BDD
cucumber/              Java — Cucumber 7, Karate 1.5, TestNG, Selenium
cucumber_python/       Python — Behave, Selenium

# API / Service
postman/               JSON/JS — Newman 6.2.1
fastapi-service/       Python — FastAPI, Redis, Pytest, k6
pact-consumer/         TypeScript — Pact v13, Vitest, pact-python verifier

# LLM Evaluation
ai-eval/               Python — DeepEval, Pytest, OpenAI, ChromaDB
conv-eval/             Python — DeepEval, Pytest, OpenAI
agent-eval/            Python — DeepEval, Pytest, OpenAI, Pydantic

# AI Agents
job-agent/             Python — Anthropic Claude, Tavily, AgentOps
coding-agent/          Python — Anthropic Claude, AgentOps
failure-triage/        Python — Anthropic Claude (tool use), JUnit XML, DataDog

# LLM Frameworks
langchain-rag/         Python — LangChain 0.3, LCEL, ChromaDB, Langfuse
langgraph-agent/       Python — LangGraph 0.4, LangChain Anthropic
dspy-optimizer/        Python — DSPy 2.6, OpenAI
dspy-vertex/           Python — DSPy 2.6, Vertex AI Gemini

# Data & Analytics
claims-diff/           Python — Pandas, Pydantic, BigQuery (optional)
flakiness-detector/    Python — JUnit XML, Click, DataDog
quality-dashboard/     Python — JUnit XML, DataDog v2 API, GitHub Actions API
site-monitor/          Python — BeautifulSoup, Click, DataDog, Requests
vulnerability-aggregator/ Python — GitHub API, Dependabot, CodeQL, ZAP
dependency-audit/      Python — Click, npm/PyPI/NuGet/Maven registries

# QMS & Compliance
qms-evidence-collector/ Python — Click, ISO 9001, SOC 2, ISO/IEC 17025, DataDog

# Infrastructure
terraform/             HCL — Terraform >= 1.6, AWS, DataDog
k8s/                   YAML — Kubernetes manifests, Selenium Grid, Healenium

# Automation (Claude Code)
automation/headless/   Bash — claude -p scripts for CI pipelines
automation/agent-sdk/  TypeScript — Agent SDK portfolio health orchestrator
automation/routines/   Markdown — Routine prompts for cloud-managed schedules

# Companion repos (external)
# legal-funding-qa-agent  — Python, LangGraph, DSPy, Hypothesis, Claude
# agentic-p2p-auditor     — Python, Claude (tool use), Decimal math
# ai-pr-reviewer          — JavaScript, Claude, promptfoo, Docker
```

## Architecture principles — follow these in all changes

### Page Object Model (POM)

Every browser framework uses POM with a shared `BasePage` superclass. Locators and page interactions live in page objects, never in test files. When adding or modifying UI interactions:

- Put locators and actions in the appropriate page class under `pages/`.
- Extend `BasePage` — use its `waitForVisibility`, `click`, `getText` utilities instead of raw driver calls.
- If a new page is needed, create a new class extending `BasePage`. One page class per logical page or major component.
- Tests call page methods; tests never reference selectors directly.

### Separation of test logic from test data

- **Cypress:** Fixtures in `cypress/fixtures/*.json`, loaded via `cy.fixture()`.
- **Java (Selenium/Cucumber):** `@DataProvider` for parametrized tests, `Examples` tables in `.feature` files, `config.properties` for env config.
- **Python (Pytest):** `@pytest.mark.parametrize` driven by JSON datasets in `datasets/`. Session-scoped fixtures in `conftest.py` for expensive resources (API clients, vector stores).
- **Python (Behave):** `Scenario Outline` with `Examples` in `.feature` files; `config.ini` for env config.
- Never hardcode test data in test methods. Extract to fixtures, data providers, or dataset files.

### Configuration management

- Environment-specific values (URLs, credentials, timeouts, browser choice) go in `.env` files or `config.properties`, never hardcoded.
- Always provide a `.env.example` when adding a new framework or new env vars.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SDETBMan/qa-automation-portfolio](https://github.com/SDETBMan/qa-automation-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
