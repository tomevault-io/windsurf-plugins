---
trigger: always_on
description: <skills_system priority="1">
---

# AGENTS

<skills_system priority="1">

## Available Skills

<!-- SKILLS_TABLE_START -->
<usage>
When users ask you to perform tasks, check if any of the available skills below can help complete the task more effectively. Skills provide specialized capabilities and domain knowledge.

How to use skills:
- Invoke: `npx openskills read <skill-name>` (run in your shell)
  - For multiple: `npx openskills read skill-one,skill-two`
- The skill content will load with detailed instructions on how to complete the task
- Base directory provided in output for resolving bundled resources (references/, scripts/, assets/)

Usage notes:
- Only use skills listed in <available_skills> below
- Do not invoke a skill that is already loaded in your context
- Each skill invocation is stateless
</usage>

<available_skills>

<skill>
<name>arch-nuxt-module-builder</name>
<description>Build and ship Nuxt modules with @nuxt/module-builder. Use when scaffolding, building, or maintaining a Nuxt module (unbuild preset, types, runtime).</description>
<location>project</location>
</skill>

<skill>
<name>arch-tsdown-monorepo</name>
<description>pnpm monorepo starter for TypeScript libraries with tsdown per package. Use when scaffolding or maintaining a multi-package TS/ESM repo with workspace deps and npm Trusted Publisher.</description>
<location>project</location>
</skill>

<skill>
<name>create-adaptable-composable</name>
<description>Create a library-grade Vue composable that accepts maybe-reactive inputs (MaybeRef / MaybeRefOrGetter) so callers can pass a plain value, ref, or getter. Normalize inputs with toValue()/toRef() inside reactive effects (watch/watchEffect) to keep behavior predictable and reactive. Use this skill when user asks for creating adaptable or reusable composables.</description>
<location>project</location>
</skill>

<skill>
<name>e2e-testing</name>
<description>>-</description>
<location>project</location>
</skill>

<skill>
<name>github-cli</name>
<description>GitHub CLI (gh) reference for repositories, issues, pull requests, Actions, projects, releases, gists, codespaces, and GitHub operations from the command line.</description>
<location>project</location>
</skill>

<skill>
<name>github-workflow</name>
<description>"Standard flow from any task source (link or description) to creating a PR: resolve task, create branch and TODO.md, wait for fixes, create PR against origin only. Use find-skills to discover data-source query methods; after confirmation save to global config so the discovery step can be skipped next time. Use when the user provides a task link/description, asks to 'follow GitHub workflow', or 'create PR from task'."</description>
<location>project</location>
</skill>

<skill>
<name>hairy</name>
<description>Hairy's {Opinionated} preferences and best practices for web development</description>
<location>project</location>
</skill>

<skill>
<name>hairy-utils</name>
<description>Comprehensive skills for working with @hairy/utils core utilities</description>
<location>project</location>
</skill>

<skill>
<name>nuxt</name>
<description>Nuxt full-stack Vue framework with SSR, auto-imports, and file-based routing. Use when working with Nuxt apps, server routes, useFetch, middleware, or hybrid rendering.</description>
<location>project</location>
</skill>

<skill>
<name>pinia</name>
<description>Pinia official Vue state management library, type-safe and extensible. Use when defining stores, working with state/getters/actions, or implementing store patterns in Vue apps.</description>
<location>project</location>
</skill>

<skill>
<name>pnpm</name>
<description>Node.js package manager with strict dependency resolution. Use when running pnpm specific commands, configuring workspaces, or managing dependencies with catalogs, patches, or overrides.</description>
<location>project</location>
</skill>

<skill>
<name>tsdown</name>
<description>Bundle TypeScript and JavaScript libraries with blazing-fast speed powered by Rolldown. Use when building libraries, generating type declarations, bundling for multiple formats, or migrating from tsup.</description>
<location>project</location>
</skill>

<skill>
<name>unocss</name>
<description>UnoCSS instant atomic CSS engine, superset of Tailwind CSS. Use when configuring UnoCSS, writing utility rules, shortcuts, or working with presets like Wind, Icons, Attributify.</description>
<location>project</location>
</skill>

<skill>
<name>vite</name>
<description>Vite build tool configuration, plugin API, SSR, and Vite 8 Rolldown migration. Use when working with Vite projects, vite.config.ts, Vite plugins, or building libraries/SSR apps with Vite.</description>
<location>project</location>
</skill>

<skill>
<name>vitepress</name>
<description>VitePress static site generator powered by Vite and Vue. Use when building documentation sites, configuring themes, or writing Markdown with Vue components.</description>
<location>project</location>
</skill>

<skill>
<name>vitest</name>
<description>Vitest fast unit testing framework powered by Vite with Jest-compatible API. Use when writing tests, mocking, configuring coverage, or working with test filtering and fixtures.</description>
<location>project</location>
</skill>

<skill>
<name>vue</name>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hairyf/vue3-pixi](https://github.com/hairyf/vue3-pixi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
