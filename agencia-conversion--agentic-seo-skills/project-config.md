---
trigger: always_on
description: Agentic SEO is a Claude Code-first plugin that should remain portable to Codex, Antigravity, and other agents that read `AGENTS.md`.
---

# Agentic SEO Agent Instructions

Agentic SEO is a Claude Code-first plugin that should remain portable to Codex, Antigravity, and other agents that read `AGENTS.md`.

## Product Direction

Agentic SEO implements Agentic SEO through six pillars:

1. Strategy
2. Brain
3. Technology
4. Technical SEO
5. Content
6. Data and Analysis

Humans own judgment. Agents execute intelligence. Agent decisions may update project files directly when evidence and checks are recorded in `project/brain/log.md`; `tipo: aprovacao` remains valid only for legacy log history.

## Repository Shape

This repository root is the plugin root.

- Claude Code manifest: `.claude-plugin/plugin.json`
- Codex manifest: `.codex-plugin/plugin.json`
- Skills: `skills/<skill-name>/SKILL.md`
- Refactor continuity: `docs/refactor-status.md`
- Tools: `tools/` for deterministic provider CLIs.
- Templates: `templates/`
- Utility scripts: `scripts/`
- Runtime project: `project/` and ignored by git except `project/.gitkeep`

## Compatibility Rules

- Keep skill bodies in standard `SKILL.md` directories so Claude Code and Codex can discover them.
- Keep cross-tool behavior in `AGENTS.md`, not only in Claude-specific files.
- Keep user-facing runtime behavior in the canonical `agentic-seo` skill; `AGENTS.md` and `CLAUDE.md` are development guidance.
- Do not rely on terminal output as the primary UX for nontechnical users.
- Prefer local web UI artifacts for previews, decisions, and reports.
- Do not commit secrets, raw user project data, generated runs, or provider responses from real clients.

## Process Integrity

The default is to follow the full documented process. Do not skip analysis, decision recording, review, lint, source separation, or other gates because the user gave a narrow request, because an old artifact exists, or because a shortcut seems sufficient.

- A process step may be skipped only when the current user explicitly asks to skip that specific step or confirms the bypass after the agent names the missing step and consequence.
- Existing drafts, previous briefings, homepage-only context, or agent confidence do not waive preconditions.
- When a bypass is explicit, record it in the artifact and append a `tipo: decisao` entry to `project/brain/log.md` before presenting the result. State clearly that the artifact is not data-backed for the skipped dimension.
- A decision on an artifact is not acceptance of an undisclosed bypass. Decision requests must show missing analysis, missing sources, and skipped checks before the user decides.
- If a required process cannot run, stop at the gate, run the local browser handoff as the agent when possible, and present only a friendly user instruction. Do not hand bash commands to the user as the UX for decisions or gates.

## Language Fidelity

Agentic SEO is English-first and supports Brazilian Portuguese as an official second language, but generated natural-language output should work in any requested language.

- Preserve the spelling, accents, and diacritics of the output language in all human-facing prose, headings, UI text, Markdown, logs, reports, prompts, and review notes.
- For pt-BR, write correct Portuguese with accents: `página`, `conteúdo`, `análise`, `evidência`, `aprovação`, `técnico`, `não`, `até`.
- ASCII transliteration is allowed only for slugs, file paths, IDs, enum values, command names, provider payloads, code identifiers, or verbatim source text that originally has no diacritics.
- Never strip accents from user-provided names, titles, claims, excerpts, anchors, or editorial text while summarizing, extracting, reviewing, or rewriting.

## Brain Rules

Every Agentic SEO project keeps authorial knowledge in one place: `project/brain/`. Open it as the Obsidian vault. Public content, evidence, and intermediate artifacts live outside `brain/` so the brain stays small and editable.

### Layout

```
project/
  brain/
    index.md          # mapa + dashboard curto
    identidade.md     # brandbook narrativo (aposto, parágrafo, frase-marca, público, canais)
    voz.md            # princípios editoriais
    tecnologia.md     # contexto técnico observado + SEO técnico
    editorial.md      # áreas de conteúdo
    topic-clusters.md # clusters semânticos
    log.md            # append-only, autoral
  sources/            # raw, imutável
  conteudos/          # produto público
    blog/<slug>.md
    linkedin/<slug>.md
    podcast/<slug>.md
    outros/<slug>.md
  artifacts/          # entregáveis intermediários e drafts
  workbench/          # rascunhos antes de virar entrada no brain
```

### Frontmatter

Brain pages: `title`, `updated`. No `status`, `judgment_level`, `pillar`, `owner`, `approved_by`, `approved_at`. Confiança vem do log.

Public content (`conteudos/<origem>/<slug>.md`): `title`, `slug`, `published_at`, `source_url`, `origem` (`blog | linkedin | podcast | outros`), `area` (slug que existe como seção em `brain/editorial.md`).

### Brain-first protocol


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agencia-conversion/agentic-seo-skills](https://github.com/agencia-conversion/agentic-seo-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
