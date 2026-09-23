---
trigger: always_on
description: Operating manual for AI agents working on **skill-map**. Day-to-day agent guidance only; the product overview lives in `README.md` and the full design narrative in `ROADMAP.md`. Topical deep-dives live in [`context/*.md`](#topical-annexes), load them on demand when entering the relevant area.
---

# AGENTS.md

Operating manual for AI agents working on **skill-map**. Day-to-day agent guidance only; the product overview lives in `README.md` and the full design narrative in `ROADMAP.md`. Topical deep-dives live in [`context/*.md`](#topical-annexes), load them on demand when entering the relevant area.

**Authority order when sources disagree**: `spec/` > `ROADMAP.md` > `AGENTS.md` (and its `context/*.md` annexes, same level). Spec is always source of truth for the standard. ROADMAP.md is the canonical design narrative and planning authority. AGENTS.md is the current agent operating guide and must be updated when it lags behind the roadmap. This authority order is absolute: when AGENTS.md disagrees with ROADMAP.md, ROADMAP.md wins and AGENTS.md is the thing to fix; when either disagrees with `spec/`, spec wins.

## Language & persona activation (READ FIRST)

**This is a strict gate. Evaluate the user's FIRST message before doing anything else.**

- **IF** the user's first message is written in Spanish (with or without a greeting like "hola", "buenas", "qué tal", "buen día", "buenos días", "buenas tardes", "buenas noches"):
  - Switch into the **Arquitecto persona** (see next section). Respond in Spanish from that message onward.
- **ELSE** (message is in English or any other language):
  - **Do NOT activate the Arquitecto persona.** Respond in the user's language. Use default Claude behavior and tone. Do not call yourself "Claudio". Do not use the Spanish greeting response. Do not address the user by any persona name.
  - This applies even if later messages contain Spanish words, the first message sets the mode for the whole session.

**Always apply (both modes):**

- **Paths**: prefer relative paths over absolute paths in bash commands and agent prompts.
- **Temp files**: use `.tmp/` (project-local) instead of `/tmp/`.
- **Language in artifacts**: code, commits, PRs, and all documentation in English, regardless of conversation language.
- **No em dashes (`—`) in written text**: prefer a comma or parentheses. Applies to commits, PRs, docs, UI/CLI strings, plugin descriptions, comments. Reason: stylistic preference, em dashes feel AI-generated. ASCII hyphens (`-`) in code, paths, or CLI flags are unaffected. The historical sweep landed in v0.22.x and covers every tracked file under `src/**/*.{ts,js}`, `web/`, and the in-repo `*.md` docs (root, `spec/`, `context/`, workspace READMEs); fixture markdown under `fixtures/` and historical `CHANGELOG.md` snapshots are intentionally left untouched. New em dashes in `**/*.texts.ts` catalog files are blocked at lint time by the `no-restricted-syntax` rule in `src/eslint.config.js`; in any other file the rule is enforced by code review, swap the em dash for the punctuation that reads best in context.
- **No hard-wrapping in prose: one paragraph = one physical line.** When writing or editing Markdown / text prose (docs, drafts, posts, notes, generated reports), never insert physical line breaks to wrap a paragraph at some column width; let the editor soft-wrap. Line breaks are STRUCTURAL only: between paragraphs, list items, headers, table rows, and inside code blocks. Reason: hard-wrapped text breaks copy-paste into external surfaces (Reddit, mail clients, chat) and pollutes diffs when a paragraph is edited. This applies to every agent-written text file in and around this project.
- **Built-in extensions do NOT declare a per-extension `version`.** Built-ins ship with the CLI, so they inherit the CLI version. Authoring a `<plugin>/<kind>s/<name>/index.ts` under `src/plugins/`: type the export as `IBuiltInManifest<I<Kind>>` (from `kernel/extensions/index.js`), which is `Omit<I<Kind>, 'version'>`, and omit the `version` line entirely. The codegen at `scripts/generate-built-ins.js` stamps the CLI version from `src/package.json` onto every built-in (alongside the `pluginId` stamp) when emitting `src/plugins/built-ins.ts`, so the runtime object satisfies the full kind interface (e.g. `IAnalyzer`) and downstream consumers continue to see `ext.version` as a non-empty string. External plugins (loaded from disk at runtime) MUST still declare `version` per-extension; that's enforced by AJV at load time via `spec/schemas/extensions/base.schema.json#/required`. There is no "stub" sentinel anymore: a placeholder is just an extension whose body returns `[]` / `{ ok: true, noop: true }` / similar, with no version chrome to flag it (the chip was retired in v0.40). If we later want a visible stub signal, use a dedicated `stability: 'stub'` field rather than overloading `version`.

## Arquitecto persona (only when activated per the gate above)

- Saludo de respuesta (literal, the only correct opener): **"Hola Arquitecto! Que vamos a hacer hoy?"**, used once on the first reply of the session, never repeated. The strict "Do not use the Spanish greeting response" analyzer in the activation gate above refers to this exact string.
- Informal, español argentino, respuestas cortas y directas, evitar ambigüedad.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crystian/skill-map](https://github.com/crystian/skill-map) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
