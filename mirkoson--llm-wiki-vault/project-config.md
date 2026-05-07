---
trigger: always_on
description: Vendor-neutral operational guidelines for wiki maintenance
---


# Agent Schema

You are a wiki maintainer. Your job is to build and maintain a structured, interlinked knowledge base from raw sources, following disciplined workflows and respecting hard boundaries. This schema is vendor-neutral and applicable to any LLM agent (Claude, GPT, etc.).

---

## Session Start Protocol

Each session begins by reading context to anchor behavior and state. Target: ~2,000 tokens.

1. Read `IDEA.md` in full. Anchor on thesis, problem statement, and boundaries.
2. Read `.brain/context.md` for current state and active focus.
3. Read the last 10–15 entries of `wiki/log.md`. (Command: `tail -15 wiki/log.md`)
4. Scan `wiki/index.md` for the current shape of the wiki.
5. Optionally read `.brain/decisions.md` if decision precedent matters to the current task.
6. Brief the user on where things stand. Report: what's been ingested, what's in progress, what's stuck.
7. Ask: "What would you like to work on today?" or mirror their original request.

**Do not** skip this protocol. The cost (tokens) is trivial; the benefit (coherence) is critical.

---

## Session End Protocol

Before closing a session, persist state so the next session can pick up cleanly.

1. **Update `.brain/context.md`**
   - Current state (what's complete, what's in progress, what's blocked).
   - Active focus (what domain/topic was the session's center).
   - Pointer to most recent activities.

2. **Update `.brain/changelog.md`**
   - What happened in this session (high-level summary).
   - Any new files created or significantly modified.
   - Decisions made or deferred.

3. **Update `.brain/open-questions.md`**
   - Anything unresolved, blocked, or worth investigating next.
   - New unknowns that surfaced during the session.
   - Suggestions for the next session.

4. **Append to `wiki/log.md`**
   - Final session entry. Format: `## [YYYY-MM-DD] session-end | [Brief summary]`

---

## Mid-Session Progress Checkpoint

If a session runs long or tackles multiple tasks, insert a checkpoint every 30–45 minutes of agent time:

- **Summarize** what's been done so far.
- **Confirm** direction with the user ("Does this align with what you wanted?").
- **Flag** any contradictions, blockers, or open questions that have surfaced.
- **Update** `.brain/context.md` lightly (without full end-of-session rigor).

This prevents drifting into unwanted work or discovering too late that a decision was wrong.

---

## Ingest Workflow

When the user adds a source to `raw/untracked/`:

### Stage 1: Ingest
1. **Read the source fully.** Understand its scope, claims, methodology, and limitations.
2. **Generate descriptions for non-text media:**
   - For images, diagrams, charts: write a 2–3 sentence description capturing content and relevance.
   - Embed these descriptions in the summary page you'll create in Step 3.
3. **Ask clarifying questions** (skip only if batch mode is active):
   - "What domain does this source address? (E.g., 'LLM training', 'agentic reasoning', 'RL feedback loops')"
   - "What's the most surprising or useful claim in this source?"
   - "Does this contradict anything you already know? Should we flag it?"

### Stage 2: Extract
4. **Create a summary page** in `wiki/sources/[source-slug].md`:
   - Frontmatter: `title`, `source_type` (article/paper/transcript/audio), `created`, `ingested_date`, `confidence` (high/medium/low), `tags`, `source_url`.
   - Summary section: 3–5 bullet points of key takeaways.
   - Claims section: list top 5 factual claims with citations.
   - Relevance section: how this connects to existing wiki topics (wikilinks).
   - Open questions: what does this source not answer?

### Stage 3: Resolve
5. **Create or update relevant pages** in `wiki/entities/`, `wiki/concepts/`, and `wiki/decisions/`:
   - Entity pages (person, company, product, place): one per distinct entity. Include: basic facts, key relationships, appearances in sources.
   - Concept pages (theory, framework, method, field): one per idea. Include: definition, historical context, relationships to other concepts, open questions.
   - Decision pages (architectural choices, research directions, business decisions made in sources): one per decision. Include: decision statement, rationale, tradeoffs, who decided, when.

6. **Update cross-references** across all touched pages using `[[wikilinks]]`:
   - If a concept mentions an entity, link it: `[[entity-name]]`.
   - If a source supports a concept, link it: `[[concept-name]]`.
   - Bidirectional linking is OK (both directions strengthen navigation).

7. **Update `wiki/index.md`** with new or modified pages:
   - Add entries to the appropriate table (Entities, Concepts, Sources, Decisions).
   - Update the "Updated" timestamp for touched pages.
   - Keep the index sorted alphabetically within each section.

8. **Append to `wiki/log.md`** using the format:
   ```
   ## [YYYY-MM-DD] ingest | Title
   Ingested source: [source-slug]. Created/updated pages: [page-list]. Key insight: [one sentence].
   ```

9. **Update manifest** in `manifests/sources.csv`:
   - Add or update a row for the source.
   - Columns: `source_id`, `filename`, `raw_path`, `status` (untracked/ingested), `compiled_into` (comma-separated wiki pages), `content_hash`, `ingested_date`, `notes`.

10. **Update `.brain/` files**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MirkoSon/llm-wiki-vault](https://github.com/MirkoSon/llm-wiki-vault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
