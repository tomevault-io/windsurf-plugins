---
trigger: always_on
description: This file gives Claude Code persistent context about this project. Keep it in sync with the actual code — run `/sync-docs` after major changes.
---

# CLAUDE.md

This file gives Claude Code persistent context about this project. Keep it in sync with the actual code — run `/sync-docs` after major changes.

## Project

- **Name**: claude-code-dashboard
- **Purpose**: Enterprise analytics dashboard for Claude Code — joins Anthropic Analytics / Admin / Compliance APIs and an S3 archive to expose adoption, productivity, cost, and audit insights with an AI natural-language query layer.
- **Stage**: Deployed (CloudFront + ALB + ECS Fargate in ap-northeast-2, account 061525506239)

## Tech Stack

| Layer | Stack |
|---|---|
| Frontend | React 18 · Vite 5 · TypeScript 5 · Tailwind 3 · Recharts 2 · React Router 6 · react-markdown 10 + remark-gfm |
| Backend | Express 4 on Node 20 · AWS SDK v3 (Bedrock Runtime, S3, Athena, Secrets Manager) |
| Infra | AWS CDK 2.170 (TypeScript) — 4 stacks (network/storage/compute/collector) |
| Runtime | Fargate ARM64 · CloudFront + WAF · ALB (CloudFront-prefix-list locked) · Secrets Manager · Lambda |
| Data | S3 NDJSON archive · Glue Data Catalog · Athena · Bedrock (Claude Sonnet 4.6 via global inference profile) |
| External APIs | Anthropic Analytics API · Admin API · Compliance API (three separate keys) |

## Project Structure

```
claude-code-dashboard/
├── src/                    React SPA (Vite)
│   ├── components/         Shared UI (Layout, ClaudeIcon, KpiCard, ChartCard, PageHeader, LoadingState, UserDetailPanel, DateRangeControl, CsvUploader, Markdown, SortableTh). Layout pins the sidebar to the viewport via h-screen + per-pane overflow-y-auto so scrolling moves only the main pane; it also renders the version badge (links to /changelog) and the static AWS run-rate label.
│   ├── pages/              14 routes (Overview, Executive, Users, UserProductivity, UserSearch, Trends, ClaudeCode, Productivity, Adoption, Cost, Compliance, Analyze, Archive, Changelog). Default date range is 7d. The DateRangeControl now allows today as the end date (Analytics 3-day buffer surfaces partial counts; the popover footnote calls this out).
│   ├── lib/                i18n (ko/en), useDateRange, useFetch, useHealth, useSortable, format (masking, number, date)
│   ├── types.ts            Analytics API schema types
│   ├── App.tsx             Router
│   ├── index.css           Tailwind entry + the generic `@media print` block keyed off `body.app-print` that powers Save-as-PDF on Analyze, Cost, and Executive (visibility-isolated `.print-export` subtree, auto-expanded `<details>`, Claude palette preserved on paper)
│   └── main.tsx            Entry + I18nProvider
├── server/                 Express API layer
│   ├── index.js            Proxy routes: /api/analytics/*, /api/admin/*, /api/compliance/* (after_id cursor; startup prewarm + 5-min interval refresh for the 7d/14d/30d audit windows), /api/health, plus a 10-minute in-memory cache shared across upstream calls
│   ├── aws.js              registerAwsRoutes(): /api/cost/{live,csv,upload,uploads,uploads/:file,efficiency}, /api/analyze (Bedrock SSE), /api/archive/query (Athena, 60-second polling budget), plus the analytics→CsvResp reshape used by /cost/live
│   └── mock.js             Deterministic mock generators (dev fallback only)
├── collector/              Node 20 Lambda — daily S3 snapshot of Analytics API
│   ├── handler.js          Flatten → NDJSON → s3://<bucket>/<table>/date=YYYY-MM-DD/
│   └── glue-schemas.md     Columnar schema for Athena
├── infra/                  CDK (TypeScript)
│   ├── bin/app.ts          Entry — 4 stacks with context-driven VPC selection
│   └── lib/                network-stack · storage-stack · compute-stack · collector-stack
├── public/claude.svg       Favicon
├── docs/                   Architecture, ADRs, runbooks, onboarding, API reference
├── scripts/                setup + install-hooks
├── tests/                  Harness tests (hooks, structure, secrets)
└── tools/prompts/          AI prompt templates
```

## Key Commands

```bash
# Local dev
npm install                    # root + infra + collector should be installed separately
npm run dev                    # Vite (5173) + Express (5174) concurrently
npm run build                  # tsc -b && vite build → dist/
npm run preview                # preview built bundle
npm run server                 # Express standalone (prod behavior)

# Infra
cd infra && npm install
npx cdk synth --context existingVpcId=vpc-0dfa5610180dfa628
npx cdk deploy --all --require-approval never --context existingVpcId=vpc-0dfa5610180dfa628
npx cdk deploy ccd-compute --context existingVpcId=vpc-0dfa5610180dfa628   # single stack

# Collector
aws lambda invoke --region ap-northeast-2 --function-name ccd-collector-Fn9270CBC0-DAPvUci8ngg6 \
  --payload '{"date":"2026-04-18"}' --cli-binary-format raw-in-base64-out /tmp/out.json
```

## Conventions

- **Language**: Korean for conversation and commit messages, English for code/identifiers/UI strings (the UI has a runtime en/ko toggle).
- **TypeScript**: strict mode, noUnusedLocals, noUnusedParameters.
- **Server code**: ESM (`"type": "module"` in package.json). Use `node --check` for syntax validation before deploy.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whchoi98/claude-code-dashboard](https://github.com/whchoi98/claude-code-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
