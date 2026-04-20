---
trigger: always_on
description: > **IMPORTANT: Never develop on `main` branch!**
---

# Vue Skills Development Guide

## Branch Strategy

> **IMPORTANT: Never develop on `main` branch!**
>
> - `main` is the **publishing branch** — it only contains released skills
> - `dev` is the **development branch** — all work happens here
> - Use the "Sync to Main" GitHub Action to publish changes from `dev` to `main`

| Branch | Purpose | Direct commits |
|--------|---------|----------------|
| `main` | Publishing (`npx skills add vuejs-ai/skills`) | Forbidden |
| `dev` | Development, tests, experiments | Via PR only |

## Development Workflow

After completing any task, run:

```bash
pnpm typecheck
```

This ensures TypeScript types are correct before committing.

## Skill Scopes

| Skill | Scope |
|-------|-------|
| **vue-best-practices** | Vue 3 with Composition API, `<script setup lang="ts">`, TypeScript, SSR. Includes render functions for cases where templates can't handle the requirement. |
| **vue-options-api-best-practices** | Vue 3 Options API style (`data()`, `methods`, `this` context). Each rule shows Options API solution only. |
| **vue-jsx-best-practices** | JSX syntax in Vue (e.g., `class` vs `className`, JSX plugin config). |
| **vue-testing-best-practices** | Testing with Vitest, Vue Test Utils, and Playwright for E2E. |
| **vue-router-best-practices** | Vue Router 4 patterns, navigation guards, route params, and route-component lifecycle interactions. |
| **vue-pinia-best-practices** | Pinia stores, state management patterns, store setup, and reactivity with stores. |
| **vue-debug-guides** | Debugging and troubleshooting Vue 3: runtime errors, warnings, async error handling, SSR hydration issues. |
| **create-adaptable-composable** | Creating reusable composables with `MaybeRef`/`MaybeRefOrGetter` input patterns. |

## Common Pitfalls & Best Practices

- **Vue-specific only:** Reference files must cover Vue patterns and gotchas, not general web/JS knowledge.
- **No edge cases:** Avoid niche scenarios, tool-specific quirks, and obvious/well-known practices.
- **Required structure:** Each reference file needs title, impact level, task checklist, and incorrect/correct code examples.
- **SKILL.md is for coding agents:** Follow [official best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices): be concise (context window is a public good), assume the coding agent is smart (only add what it doesn't know), no hardcoded counts or historical background. Use progressive disclosure — SKILL.md is an overview that points to reference files.

## Checklist for effective Skills

Before sharing a Skill, verify:

### Core quality
- [ ] Description is specific and includes key terms
- [ ] Description includes both what the Skill does and when to use it
- [ ] SKILL.md body is under 500 lines
- [ ] Additional details are in separate files (if needed)
- [ ] No time-sensitive information (or in "old patterns" section)
- [ ] Consistent terminology throughout
- [ ] Examples are concrete, not abstract
- [ ] File references are one level deep
- [ ] Progressive disclosure used appropriately
- [ ] Workflows have clear steps

### Code and scripts
- [ ] Scripts solve problems rather than punt to Claude
- [ ] Error handling is explicit and helpful
- [ ] No "voodoo constants" (all values justified)
- [ ] Required packages listed in instructions and verified as available
- [ ] Scripts have clear documentation
- [ ] No Windows-style paths (all forward slashes)
- [ ] Validation/verification steps for critical operations
- [ ] Feedback loops included for quality-critical tasks

## Evaluating Skills

### Evaluation-Driven Development

Following [Anthropic's best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices#evaluation-and-iteration):

1. **Create evaluations FIRST** - Before writing extensive documentation
2. **Establish baseline** - Run without skill, document failures
3. **Test with skill** - Run with skill loaded, compare improvement
4. **Iterate** - Refine skill based on evaluation results

### Eval Structure

Each reference file requires 3 evals organized as scenarios:

```
evals/suites/skills/vue-best-practices/
├── computed-vs-methods/           # Reference name
│   ├── scenario-1/                # Eval 1 of 3
│   │   ├── eval.json
│   │   ├── eval.ts
│   │   ├── results.json           # Tracks run results
│   │   └── src/...
│   ├── scenario-2/                # Eval 2 of 3
│   └── scenario-3/                # Eval 3 of 3
```

Each scenario is a self-contained Vue project:

```
evals/suites/skills/vue-best-practices/computed-vs-methods/scenario-1/
├── eval.json           # Eval configuration with query and expected behavior
├── eval.ts             # Vitest assertions (withheld from agent)
├── results.json        # Tracks run results per model
├── package.json        # Self-contained Vue project
├── vite.config.ts
├── tsconfig.json
├── index.html
├── eslint.config.js
└── src/
    ├── main.ts
    ├── App.vue
    ├── vite-env.d.ts
    └── components/
        └── ProductList.vue   # Empty stub component
```

**Key files:**
- `eval.json` - Eval configuration with query, skills, and expected_behavior
- `eval.ts` - Vitest tests that validate the generated code (withheld from agent during generation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vuejs-ai/skills](https://github.com/vuejs-ai/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
