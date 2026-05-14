---
trigger: always_on
description: Technical reference for developers and AI agents working on this repo.
---

# CLAUDE.md — AgentKit Repository Guide

Technical reference for developers and AI agents working on this repo.

---

## What is AgentKit?

AgentKit is an open-source repository of ready-to-deploy AI agent projects built on [Lamatic](https://lamatic.ai) flows. Each contribution is a self-contained folder under `kits/` that you can fork, configure, and deploy. Lamatic Studio exports directly in the format documented here.

---

## Repository Structure

Flat — every contribution lives under `kits/<name>/`. No categories, no separate `bundles/` or `templates/` directories.

```
AgentKit/
├── kits/                  # All contributions (flat)
│   ├── <name>/            # Each kit/bundle/template
│   ├── <name>/
│   └── ...
├── public/                # Shared static assets
├── .github/               # GitHub workflows, PR templates
├── CONTRIBUTING.md        # Contributor guide (read first)
├── CLAUDE.md              # This file
├── README.md
├── CODE_OF_CONDUCT.md
├── CHALLENGE.md
└── LICENSE
```

---

## The Three Types

All three live under `kits/` and are distinguished by the `type` field in `lamatic.config.ts` and by which directories are present.

| Type | `type` value | `apps/` | flows | Purpose |
|---|---|:---:|---|---|
| **Template** | `"template"` | ❌ | 1 | A single flow, no UI, no env vars. |
| **Bundle** | `"bundle"` | ❌ | 2+ | Multiple related flows, no UI. |
| **Kit** | `"kit"` | ✅ | 1+ | Flows + a runnable Next.js app. |

---

## Per-Kit Layout

```
kits/<name>/
├── lamatic.config.ts         # REQUIRED: project metadata (name, type, author, steps, links)
├── agent.md                  # REQUIRED: LLM-generated agent identity + capability doc
├── README.md                 # REQUIRED: human-readable setup guide
├── .gitignore
│
├── flows/                    # REQUIRED: flat .ts files, one per flow
│   └── <flow-name>.ts        # Self-contained: meta + inputs + references + nodes + edges
│
├── constitutions/            # REQUIRED: guardrails / behavioral rules
│   └── default.md
│
├── prompts/                  # OPTIONAL: externalized LLM prompts (system/user/assistant)
│   └── <flow>_<node>_<role>.md
│
├── scripts/                  # OPTIONAL: externalized code from codeNode nodes
│   └── <flow>_<node>.ts
│
├── model-configs/            # OPTIONAL: externalized LLM/RAG/ImageGen model settings
│   └── <flow>_<node>.ts
│
├── triggers/                 # OPTIONAL: widget UI settings (chat/search appearance)
│   └── widgets/<flow>_<node>.ts
│
├── memory/                   # OPTIONAL: memory node configs
│   └── <flow>_<node>.ts
│
├── tools/                    # OPTIONAL: tool ID arrays referenced by nodes
│   └── <flow>_<node>_tools.ts
│
├── apps/                     # KIT-ONLY: the runnable Next.js app
│   ├── package.json, next.config.mjs, tsconfig.json
│   ├── app/ | components/ | hooks/
│   ├── actions/orchestrate.ts
│   ├── lib/lamatic-client.ts
│   └── .env.example
│
└── assets/                   # OPTIONAL: static images/documents/data used by flows
```

---

## `lamatic.config.ts` — Metadata

```typescript
export default {
  name: "Project Name",
  description: "Short description.",
  version: "1.0.0",
  type: "template" as const,          // "kit" | "bundle" | "template"
  author: { name: "...", email: "..." },
  tags: ["lowercase", "plain", "strings"],
  steps: [
    { id: "flow-name", type: "mandatory" as const, envKey: "FLOW_ENV_KEY" }
  ],
  links: {
    demo: "https://...",              // optional
    github: "https://github.com/Lamatic/AgentKit/tree/main/kits/<name>",
    deploy: "https://vercel.com/new/clone?...",   // kits only; root-directory=kits/<name>/apps
    docs: "https://lamatic.ai/docs/..."
  }
};
```

Step kinds:
- `"mandatory"` — a required flow, typically has `envKey` for kits.
- `"any-of"` — onboarding choice, has `options[]`, `minSelection`, `maxSelection`. Can declare `prerequisiteSteps`.

---

## Flow `.ts` File Structure

Each flow is ONE self-contained TypeScript file. Flows may have a top-of-file block comment containing human+agent-readable documentation (embedded from the old `flows.md`).

```typescript
/*
 * # Flow Name
 * Description of what this flow does.
 * ... (embedded documentation from LLM-generated flow.md)
 */

// Flow: <flow-name>

export const meta = {
  name: "...", description: "...", tags: [...], testInput: {...}, author: {...}
};

export const inputs = {
  // Per-node input schema (what Lamatic Studio treats as privately-configured fields)
  "NodeId_xxx": [ { name, label, type, required, isPrivate, ... } ]
};

export const references = {
  // Cross-references to externalized resources this flow depends on
  prompts:        { <camelCaseKey>: "@prompts/<file>.md" },
  scripts:        { <camelCaseKey>: "@scripts/<file>.ts" },
  modelConfigs:   { <camelCaseKey>: "@model-configs/<file>.ts" },
  constitutions:  { default: "@constitutions/default.md" },
  triggers:       { <camelCaseKey>: "@triggers/widgets/<file>.ts" },
  memory:         { <camelCaseKey>: "@memory/<file>.ts" },
  tools:          { <camelCaseKey>: "@tools/<file>.ts" }
};

export const nodes = [ /* exact Lamatic Studio node graph */ ];
export const edges = [ /* exact Lamatic Studio edge list */ ];

export default { meta, inputs, references, nodes, edges };
```

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lamatic/AgentKit](https://github.com/Lamatic/AgentKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
