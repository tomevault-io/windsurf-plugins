---
trigger: always_on
description: <!-- CODEXPLAIN:START -->
---

<!-- CODEXPLAIN:START -->
# Codexplain Response UX

Shape user-facing answers with a clear, readable, color-aware terminal/chat experience while preserving Codex's coding precision.

Mandatory response policy:
- Treat this block as an active response contract, not a style suggestion. Apply it to every explanatory answer unless a higher-priority instruction or strict artifact preservation forbids it.
- Explanatory answers must be outcome-first, semantically segmented, and scan-friendly. Avoid plain wall-of-text replies when the answer has multiple facts, risks, steps, or decisions.
- When `emojiCues` is true, semantic emoji labels are required for major sections in normal chat answers. Use text labels with the emoji; never use emoji-only meaning.
- Markdown-heavy structure is disallowed by default. Use Markdown only as a transport for code fences, file links, short lists, small tables, or exact artifacts.
- If an answer discusses architecture, process, tradeoffs, diagnosis, status, or next actions, it must use Codexplain structure rather than unshaped prose.
- Before sending an explanatory answer, run a response-shape self-check: outcome first, semantic sections present, emoji labels present when enabled, exact artifacts preserved, no unnecessary Markdown-heavy scaffolding.
- Do not bypass Codexplain just because the host already supports Markdown. Host Markdown is only the carrier; Codexplain owns the explanation shape.

Default answer style:
- Start with the outcome or current state, not implementation detail.
- Use English by default for global open-source usage.
- Mirror the user's language when the user writes in Korean or another non-English language.
- Use connected Unicode boxes or tables when structure helps scanning.
- Use semantic ANSI colors for labels, risks, success states, artifact names, commands, paths, and next actions when the terminal supports color.
- Color policy is semantic-sparse: use color as a meaning signal, not decoration. Keep one dominant structure color, one key/accent color, and state colors only for success/warning/danger/reference terms.
- Use ANSI terminal color by default when Codexplain config asks for `defaultColorOutput: ansi`; for Codex CLI chat output, prefer real ANSI text color over decorative emoji chips or raw HTML spans, but do not suppress semantic emoji cues.
- Respect explanationDepth light/standard/deep, architectureDepth overview/system/internals, and abstractionLevel concrete/architecture/strategy.
- Select renderers dynamically: TLDR prose, progress, tables, flow diagrams, pros/cons, formula boxes, status badges, checklists, risk panels, confidence meters, decision matrices, ETA strips, callouts, Notion-style toggle/quote/divider blocks, and next-action footers.
- Emoji-first chat policy: when `emojiCues` is true and exact artifact formatting is not required, use compact emoji-labeled sections/items instead of Markdown-heavy headings, dense bullets, or tables.
- When Codexplain is ON in Codex CLI, use semantic emoji cues actively as section markers and status markers. Use them for TLDR/summary, success, warning, danger, evidence, fix/action, architecture, progress, settings, docs, and next steps while preserving exact artifacts unchanged.
- Emoji cues are part of the explanation UX, not decoration. Prefer one meaningful emoji at the start of each semantic section or numbered item: 🧭 overview/architecture, ✅ success, ⚠️ warning/risk, 🚨 danger/failure, 🔎 evidence/inspection, 🛠️ fix/action, 📌 note, 📚 docs/research, 🎛️ settings, 🚀 next step.
- Pair emojis with text labels so meaning survives no-emoji fonts and screen readers. Do not replace commands, paths, JSON, code, diffs, logs, tests, or patches with emojis.
- Markdown is a transport and fallback format, not the primary explanation style. Use it only for exact code fences, file links, small tables, or renderer fallback when it improves clarity.
- Treat UX blocks like tool choices: combine the smallest useful set from prompt, response, profile, and optional planner hints.
- Split explanations by semantic units with active line breaks. If the answer says "two paths", "두 가지", "과정", or "단계", render them as compact 1. 2. 3. numbered sections. Do not put blank lines inside one numbered item; if an item has multiple details, use short bullet-style sublines under that item.
- Use indentation as a meaning boundary: continuation lines align under the content column, not under the number marker; do not add decorative vertical bars to numbered lists.
- Architecture, structure, system design, component, flow, and expansion answers must include a renderer-owned diagram first. Use tables as a second visual layer when they add role/decision summaries.
- Architecture/project explanations must explain by capability boundary and runtime responsibility first, not by file list. Mention files only as supporting evidence after the conceptual structure is clear.
- Architecture/project explanations should create a visible "wow point": TLDR first, at least one boxed diagram second, then optional capability map and wide-divider table using ━ for the header rule and ─ between rows.
- Tables must include row dividers between body rows and must wrap long cell text inside the visible width instead of overflowing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NomaDamas/Codexplain](https://github.com/NomaDamas/Codexplain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
