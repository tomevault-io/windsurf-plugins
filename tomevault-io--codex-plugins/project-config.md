---
trigger: always_on
description: SEO Brain is a Claude Code-first plugin that should remain portable to Codex, Antigravity, and other agents that read `AGENTS.md`.
---

# SEO Brain Agent Instructions

SEO Brain is a Claude Code-first plugin that should remain portable to Codex, Antigravity, and other agents that read `AGENTS.md`.

## Product Direction

SEO Brain implements Agentic SEO through six pillars:

1. Strategy
2. LLM Wiki
3. Technology
4. Technical SEO
5. Content
6. Data and Analysis

Humans own judgment. Agents execute intelligence. A draft created by an agent is not approved strategic context until the user explicitly approves it.

## Repository Shape

This repository root is the plugin root.

- Claude Code manifest: `.claude-plugin/plugin.json`
- Codex manifest: `.codex-plugin/plugin.json`
- Skills: `skills/<skill-name>/SKILL.md`
- Shared skill references: `skills/_shared/references/`
- Templates: `templates/`
- Utility scripts: `scripts/`
- Runtime project: `project/` and ignored by git except `project/.gitkeep`

## Compatibility Rules

- Keep skill bodies in standard `SKILL.md` directories so Claude Code and Codex can discover them.
- Keep cross-tool behavior in `AGENTS.md`, not only in Claude-specific files.
- Keep user-facing runtime behavior in the canonical `seo-brain` skill; `AGENTS.md` and `CLAUDE.md` are development guidance.
- Do not rely on terminal output as the primary UX for nontechnical users.
- Prefer local web UI artifacts for previews, approvals, and reports.
- Do not commit secrets, raw user project data, generated runs, or provider responses from real clients.

## Process Integrity

The default is to follow the full documented process. Do not skip analysis, approval, review, lint, source separation, or other gates because the user gave a narrow request, because an old artifact exists, or because a shortcut seems sufficient.

- A process step may be skipped only when the current user explicitly asks to skip that specific step or confirms the bypass after the agent names the missing step and consequence.
- Existing drafts, previous briefings, homepage-only context, or agent confidence do not waive preconditions.
- When a bypass is explicit, record it in the artifact and log before presenting the result. State clearly that the artifact is not data-backed for the skipped dimension.
- Approval of an artifact is not approval of an undisclosed bypass. Approval requests must show missing analysis, missing sources, and skipped checks before the user decides.
- If a required process cannot run, stop at the gate, run the local browser handoff as the agent when possible, and present only a friendly user instruction. Do not hand bash commands to the user as the UX for approvals or gates.

## Language Fidelity

SEO Brain is English-first and supports Brazilian Portuguese as an official second language, but generated natural-language output should work in any requested language.

- Preserve the spelling, accents, and diacritics of the output language in all human-facing prose, headings, UI text, Markdown, logs, reports, prompts, and review notes.
- For pt-BR, write correct Portuguese with accents: `página`, `conteúdo`, `análise`, `evidência`, `aprovação`, `técnico`, `não`, `até`.
- ASCII transliteration is allowed only for slugs, file paths, IDs, enum values, command names, provider payloads, code identifiers, or verbatim source text that originally has no diacritics.
- Never strip accents from user-provided names, titles, claims, excerpts, anchors, or editorial text while summarizing, extracting, reviewing, or rewriting.

## Wiki Rules

Every SEO Brain project should use Obsidian-compatible Markdown and separate sources from synthesis.

- Raw sources live in `sources/` and should be treated as immutable or append-only.
- Generated and curated knowledge lives in `wiki/`; open `project/wiki/` as the Obsidian vault.
- `wiki/fontes/index.md` is a catalog of raw evidence, but the raw files themselves remain in `sources/`.
- Use Obsidian wikilinks only for real pages inside `wiki/`; use normal Markdown links for files under `../sources/`.
- Strategic pages require explicit human approval.
- Operational and observational pages may be updated by agents when checks pass.
- Important events must be appended to `wiki/log/index.md`. Each entry must declare a `type` of `strategic-approval` or `operational-decision` so events can be filtered by audience.
- The wiki never holds drafts or hypotheses. Pages either reflect approved/measured state or do not exist yet.
- `project/workbench/` is only for construction: research, briefing, auxiliary analysis, and intermediate context.
- Complete deliverables, including v0 artifacts, live in `project/artifacts/`; content drafts live in `project/artifacts/contents/<slug>/`.
- Public content also lives in `project/wiki/conteudos/` only after final approval and `status: published`.
- Hypothetical or unverified strategic work stays outside the Wiki until explicit human approval; operational pages may be promoted only after automated checks pass.

Required strategic approval pages:

- `wiki/index.md`
- `wiki/eeat.md`
- `wiki/tecnologia/index.md`
- `wiki/tom-de-voz/index.md`

## Browser Handoff

For previews, approvals, sensitive input, and option selection, prefer a local browser handoff over terminal interaction.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomevault-io/codex-plugins](https://github.com/tomevault-io/codex-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
