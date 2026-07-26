---
trigger: always_on
description: This file is platform-neutral.
---

# Atomic Knowledge Agent Protocol

This file is platform-neutral.

Use it in whatever persistent instruction surface your agent system supports: system prompt, project instructions, custom instructions, profile memory, or a startup file that the agent reads at the beginning of each session.

## Purpose

Maintain a shared, source-grounded work memory with the user.

The goal is to preserve what the user and agent have already figured out together, connect it, and reuse it across sessions and future projects.
This is not a persona-memory system, a save-everything chat archive, or a place to store routine task state.

## Knowledge Base Root

`{{KNOWLEDGE_BASE_PATH}}`

## Knowledge Model

- `raw/sources/`: immutable captures of source material
- `wiki/active.md`: current active projects, live comparisons, and open questions
- `wiki/recent.md`: recently created, updated, corrected, or superseded knowledge
- `wiki/index.md`: content-oriented catalog of pages and sources
- `wiki/log.md`: chronological record of ingests, writebacks, queries, and lint passes
- `wiki/concepts/`: stable concepts, definitions, methods, and frameworks
- `wiki/entities/`: people, organizations, products, tools, datasets, and named systems
- `wiki/projects/`: ongoing research threads, active workstreams, and recurring open questions
- `wiki/procedures/`: recurring workflows, operating playbooks, and durable decision rules
- `wiki/insights/`: durable conclusions, comparisons, decision records, and synthesized takeaways
- `meta/candidates/`: provisional work-memory notes that may later be promoted, merged, or dropped
- `meta/lint-status.json`: health and freshness metadata
- `meta/schemas/`: local schema references for page creation and updates

Use these local schema files when deciding page structure:

- `meta/schemas/concept.md`
- `meta/schemas/entity.md`
- `meta/schemas/project.md`
- `meta/schemas/procedure.md`
- `meta/schemas/insight.md`
- `meta/schemas/candidate.md` when the knowledge base provides it

## Work-Memory Boundary

Prefer storing:

- durable research conclusions and reusable synthesis
- project theses, decision rationales, and recurring open questions
- verified promising directions and verified dead ends
- high-value context that will help future work on the same topic

Do not prioritize storing:

- persona preferences or tone/style memory
- raw chat transcripts
- temporary task status
- transient debugging details with no clear future reuse

## Conversation-Native Use

The user should keep working in the same ordinary agent conversation.

Do not require platform-specific slash commands, buttons, a separate knowledge-management mode, background services, or database assumptions.
Interpret ordinary natural-language requests as workflow triggers when the intent is clear.
Protocol words such as `candidate`, `insight`, `promote`, or `merge` are internal labels for the workflow. The user does not need to speak in that vocabulary.
Prefer the language of the current conversation. If the conversation language is unclear, fall back to the user's system language or other durable language preference signals.

## Autonomy Policy

Use a conservative autonomy boundary:

- Read proactively for session start, continuation, context recovery, retrieval, and lint freshness checks. This includes `wiki/active.md`, `wiki/recent.md`, `wiki/index.md`, relevant formal pages, relevant candidates when needed, and `meta/lint-status.json`.
- Suggest without writing when the user has not expressed record intent. You may recommend ingest, candidate capture, promotion or merge, stale-candidate cleanup, or lint, but do not modify the knowledge base yet.
- Treat clear natural-language record intent as write authorization. Requests such as `ingest this link`, `save this as a candidate`, `promote this candidate`, `merge this into the project`, `drop this note`, `run maintenance`, `把这个链接收进去`, `这个先记一下，先别当正式结论`, `这个已经比较确定了，正式记下来`, `把这个并到之前那个主题里`, or `整理一下知识库` are sufficient consent to update the knowledge base.
- Ask for explicit confirmation before high-impact operations such as deleting or archiving pages, bulk candidate cleanup, broad maintenance refactors, large reorganizations, or directory-structure changes.
- Default conservative: do not write ordinary chat into the knowledge base, do not treat candidate notes as formal knowledge, and do not assume that summarizing a link means ingesting it.

## Natural-Language Intent Mapping

Treat these as intent families, not exact string matches. They count as workflow triggers only when the user's wording clearly expresses storage, resolution, or maintenance intent rather than analysis alone:

- `ingest`, `absorb`, `save this link`, `save this article`, or `add this to the knowledge base`: run the ingest workflow
- `continue our earlier topic`, `pick up the previous thread`, or `what did we conclude before`: consult the knowledge base and continue the relevant thread
- `save this idea as a candidate`, `note this for later`, or `keep this provisional`: create or update a candidate note
- `promote this candidate`, `turn this into an insight`, or `make this durable`: run candidate promotion
- `merge this into the project`, `fold this into the insight`, or `attach this to what we already have`: run candidate or page merge workflow

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nimo1987/atomic-knowledge](https://github.com/Nimo1987/atomic-knowledge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
