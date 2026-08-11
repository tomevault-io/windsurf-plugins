---
trigger: always_on
description: Project-level instructions for Claude Code sessions working on plato.
---

# CLAUDE.md

Project-level instructions for Claude Code sessions working on plato.

## Project overview

plato is an Open Source, AI-powered [microlearning](https://philosophers.group/platos-microlearning/) platform. Learners work through focused lessons in a continuous conversation with an AI coach, designed for completion in ~20 minutes.

- `client/` — React 19 + Vite SPA
- `server/` — Node.js + Hono, deployed as AWS Lambda (SAM)
- Brand: "plato" (always lowercase)

## Architecture

Quick map of the system. **Deep dives, incident history, and the *why* behind these invariants live in [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)** — read the relevant section before changing one of these subsystems.

- Login required, all data server-side. JWT access (15 min) + refresh (30 day) tokens in localStorage (`plato_auth`); login accepts email or username. Unique editable `username` per user.
- 2 Lambda functions (API Gateway buffered CRUD + Function URL streaming SSE for AI chat); 5 DynamoDB tables (users, invites, refresh-tokens, sync-data, audit-log).
- Content is `_system` sync-data (`prompt:*`, `lesson:*`, `knowledgeBase`, `settings`). Prompts are bundled in `client/prompts/*.md` and upserted on every server startup — admins can't edit prompts directly. User-created lessons live under the user's sync-data (`lessons:custom-*`).
- **9 AI agents** (prompts in `client/prompts/`, each with an HTML header documenting reads/callers/purpose): coach, lesson-creator, lesson-owner, lesson-extractor, knowledge-base-editor, knowledge-base-extractor, learner-profile-owner, learner-profile-update, course-progress-update. Program KB + Lesson Catalog are appended to system prompts at runtime (`client/js/orchestrator.js`).
- **One open source model for all 9 agents** — `qwen.qwen3-vl-235b-a22b` (Apache-2.0), declared once as `LLM` in `server/src/lib/ai-provider.js`, mirrored in `client/js/api.js`, injected into plugins as `ctx.LLM`. **No per-agent routing.** Bedrock is the only backend (local dev included — the Anthropic API provider was removed). Non-Anthropic models require the **Converse API**: legacy `InvokeModel` doesn't reject them, it returns HTTP 200 with an OpenAI-shaped body, so callers silently read `undefined`. `server/src/lib/converse.js` translates between plato's internal Anthropic wire format and Converse. A replacement model must clear **four** bars: on Bedrock us-east-2 (UIC requirement), accepts images (learners paste screenshots), reliably emits plato's literal tags, and carries an **OSI-approved license**. That last one is a constraint, not a preference — plato is AGPL-3.0, so *open weight isn't enough*: Llama 4 and Gemma ship under non-OSI community licenses and are ineligible despite passing every functional check (Llama 4 Maverick measured faster and cheaper, and is the fallback only at the cost of that bar). **Prompt caching is Claude-only on Bedrock — don't add it.** → [details](docs/ARCHITECTURE.md#ai-provider--model-choice), [why this model](docs/MODEL_SELECTION.md)
- **Conversation & image persistence** — chat history is one append-only `messages:<lessonId>` record; progress is a separate `lessonKB:<lessonId>`. **Images are never inlined in `messages:`** — each is its own compressed `screenshot:*` record, referenced by `metadata.imageKeys` (DynamoDB's 400 KB item cap; inlining base64 silently lost conversations, #191/#193). The `LessonChat` resume effect keys on stable values, never the `lesson` object identity. → [details](docs/ARCHITECTURE.md#image--conversation-persistence-191-193)
- **Link attachments** — a learner can attach a web page to a coach message. It's fetched + read **server-side** (`POST /v1/links/fetch`); the page text is injected into the coach call on that turn only (image parity), and only `{ url, title }` is persisted in `metadata.links`. SSRF defense (`server/src/lib/url-guard.js`) is load-bearing — the server fetches user URLs from inside AWS. → [details](docs/ARCHITECTURE.md#link-attachments)
- **Lessons** — three statuses: `public`, `private` (visible to `sharedWith`), `draft` (admin-only, no markdown yet; true draft iff `status==='draft'` AND markdown empty). New lessons start as `draft`; "Create Lesson" flips `draft`→`private`. Editing is conversation-based (no raw markdown editor), deep-linked at `/plato/lessons/:lessonId/edit`, with a manually-refreshed markdown preview pane. An optional `## Coach Directive` section carries author-supplied runtime instructions for the coach (e.g. "reference the learner's project", "share this code") — extracted verbatim, parsed into `lesson.coachDirective`, and surfaced via `buildContext`; it never overrides completion semantics. Optional `course` taxonomy (`_system:course:<id>`) for grouping; delete cascades `course:null`. A per-learner `courseProgress:<courseId>` record carries a tiny distilled note of what the learner demonstrated in a course's *other* lessons — regenerated on completion by the `course-progress-update` agent (incremental, ~600-char cap) and injected into the coach context as `course.progress`; informational only, never overrides completion. → [details](docs/ARCHITECTURE.md#course-progress-cross-lesson-memory)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1111philo/plato](https://github.com/1111philo/plato) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
