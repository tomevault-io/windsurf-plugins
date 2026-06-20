---
trigger: always_on
description: >
---


# sf-multiframework

Salesforce **Multi-Framework** lets you build modern frontend apps — currently **React** — that run on the Agentforce 360 Platform via the `UIBundle` metadata type. This skill is the code-first authoring path: scaffolding a project, wiring Data SDK + GraphQL, integrating the Agentforce Conversation Client, styling, and deploying.

> **Sources.** Built from the official [Salesforce developer documentation](references/official-sources.md) (the `reactdev-*` pages under `developer.salesforce.com/docs/platform/einstein-for-devs` and `code-builder`) and code patterns distilled from [`trailheadapps/multiframework-recipes`](https://github.com/trailheadapps/multiframework-recipes). Format, rubric structure, and cross-skill orchestration conventions inspired by [Jag Valaiyapathy's SF Skills](https://github.com/Jaganpro). Written and maintained by Dylan Andersen.

> **Usable in any MCP-capable agent or IDE.** Agentforce Vibes is *one* authoring surface; this skill is designed for developers who work directly against the `sf` CLI and the Data SDK without requiring a specific assistant.

> Status: **Generally available as of June 3, 2026** · API v67.0+. Runs across **all org editions — Developer Edition, Sandbox, and Production** (scratch orgs supported for development). English (`en_US`) default language required. More capabilities (incl. Agentforce Vibes 2.0) are on the way.

> Start here: [references/activation-checklist.md](references/activation-checklist.md)
> New to the feature? Read [references/overview.md](references/overview.md) then [references/setup.md](references/setup.md).

---

## When This Skill Owns the Task

Use `sf-multiframework` when the work involves:

- creating or editing files inside `force-app/main/.../uiBundles/<appName>/`
- authoring `ui-bundle.json` and `.uibundle-meta.xml`
- scaffolding via `sf template generate ui-bundle` (`reactbasic` / `default`; older Beta docs may mention `reactinternalapp` / `reactexternalapp`)
- using `@salesforce/sdk-data` (`createDataSDK`, `gql`, `dataSdk.graphql?.()`, `dataSdk.fetch?.()`)
- generating GraphQL types via codegen (`schema.graphql`, `graphql-operations-types.ts`)
- embedding the Agentforce Conversation Client (`@salesforce/agentforce-conversation-client`)
- choosing styling (SLDS blueprints vs `@salesforce/design-system-react` vs Tailwind/shadcn)
- deploying React UI bundles via `sf project deploy start --source-dir <bundle>`
- comparing React (Multi-Framework) vs LWC for a given use case

Delegate elsewhere when:

- task is pure LWC authoring → [generating-lwc-components](https://github.com/forcedotcom/sf-skills/tree/main/skills/generating-lwc-components) (older `sf-skills`: `sf-lwc`)
- writing/maintaining Apex called from the React app → [generating-apex](https://github.com/forcedotcom/sf-skills/tree/main/skills/generating-apex) (older `sf-skills`: `sf-apex`)
- the Conversation Client target is a Builder-managed Employee Agent and the work is on the agent itself → [developing-agentforce](https://github.com/forcedotcom/sf-skills/tree/main/skills/developing-agentforce) (older `sf-skills`: `sf-ai-agentforce` / `sf-ai-agentscript`)
- agent testing → [testing-agentforce](https://github.com/forcedotcom/sf-skills/tree/main/skills/testing-agentforce) (older `sf-skills`: `sf-ai-agentforce-testing`)
- generic deployment troubleshooting unrelated to UI bundle specifics → [deploying-metadata](https://github.com/forcedotcom/sf-skills/tree/main/skills/deploying-metadata) (older `sf-skills`: `sf-deploy`)

---

## Required Context to Gather First

Before authoring or fixing anything, infer or ask:

1. **Org type**: DE, Sandbox, Production, or Scratch? (All editions are supported; scratch orgs are typical for development.)
2. **App target**: `CustomApplication` (internal employee app) or `Experience` (external B2B/B2C portal)?
3. **Template**: `reactbasic`, `default`, or no template (manual setup)? Only use legacy names like `reactinternalapp` if `sf template generate ui-bundle --help` lists them.
4. **ACC needed?** If yes, is the agent an Employee Agent? Are cookies + Trusted Domains configured?
5. **Data access pattern**: GraphQL (preferred), UI API REST, or Apex REST?
6. **Styling system**: SLDS blueprints, `design-system-react`, or Tailwind/shadcn?
7. **Default language is `en_US`?** If not, scratch org def must set it.
8. **Shared storage/backends?** If multiple UI bundles reuse the same Apex or custom object, define an explicit app discriminator up front (for example `Source_App__c`) and enforce it on every read/write path.

---

## Activation Checklist

Verify these before authoring or fixing any Multi-Framework app:

1. **Org has Salesforce Multi-Framework enabled** in Setup. (Once enabled, **cannot be disabled**.)
2. **Salesforce CLI is current** and `@salesforce/plugin-ui-bundle-dev` is installed:
   `sf plugins install @salesforce/plugin-ui-bundle-dev`
3. **Node.js v22+** and **npm** installed.
4. **`sfdx-project.json` uses `sourceApiVersion: "67.0"` or higher**. The `uiBundle` field on `CustomApplication` does not exist in v66.0.
5. **App lives under `uiBundles/<appName>/`** with both `ui-bundle.json` and `<appName>.uibundle-meta.xml`.
6. **Internal apps include a companion `applications/<AppName>.app-meta.xml`** with `<uiBundle><appName></uiBundle>`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dylandersen/sf-multiframework](https://github.com/dylandersen/sf-multiframework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
