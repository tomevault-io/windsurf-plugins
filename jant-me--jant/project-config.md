---
trigger: always_on
description: Jant (short for Jantelagen) is a personal microblogging system — self-hosted, single-author, and stripped of all social mechanics. No followers, no likes, no algorithmic feed. It combines Tumblr-style multi-format posts (notes, links, quotes), Threads-style threading for connected thoughts, and curated Collections to organize content by topic. Just a clean space for one person to think out loud.
---

# Jant - Development Guide

## What is Jant

Jant (short for Jantelagen) is a personal microblogging system — self-hosted, single-author, and stripped of all social mechanics. No followers, no likes, no algorithmic feed. It combines Tumblr-style multi-format posts (notes, links, quotes), Threads-style threading for connected thoughts, and curated Collections to organize content by topic. Just a clean space for one person to think out loud.

It runs on Cloudflare Workers with minimal infrastructure. The UI follows an "Organic Minimalism" aesthetic: generous whitespace, single-column layout, smooth animations, mobile-first. Content comes in three formats — Note (), Link (shared reference), Quote (cited text) — organized through Threads and Collections.

The project is in **pre-1.0 development**. Breaking changes are expected and welcome when they improve the design. Always follow best practices over minimal-change conservatism. Update all references in the same change and document what changed in the commit message.

## Workflow Orchestration

### 1. Plan Mode Default

– Enter plan mode for ANY non-trivial task (3+ steps)
– If something goes sideways, STOP and re-plan immediately
– Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy

– Use subagents liberally to keep main context window clean
– One task per subagent for focused execution
– For complex problems, throw more compute at it

### 3. Self-Improvement Loop

– After ANY correction: update tasks/lessons.md
– Write rules that prevent the same mistake
– Ruthlessly iterate until mistake rate drops

### 4. Verification Before Done

– Never mark a task complete without proving it works
– Ask yourself: “Would a staff engineer approve this?”
– Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)

– Pause and ask “is there a more elegant way?”
– Skip this for simple fixes — don’t over-engineer

### 6. Autonomous Bug Fixing

– When given a bug report: just fix it
– Zero context switching required from the user

## Task Management

1. Plan First: Write plan to tasks/todo.md
2. Verify Plan: Check in before starting
3. Track Progress: Mark items complete as you go
4. Explain Changes: High-level summary at each step
5. Document Results: Add review section to todo.md
6. Capture Lessons: Update lessons.md after corrections

## Development Philosophy

These principles explain _why_ the codebase is structured the way it is. When you encounter a situation not covered by a specific rule, use these to guide your judgment.

- **Challenge before complying**: when the user proposes an approach that conflicts with best practices or this document, push back with a clear explanation of the trade-offs and ask for confirmation before proceeding. Silently following a suboptimal instruction is worse than a brief discussion.

- **Separation of concerns**: routes handle HTTP, services own business logic and all DB access, UI renders data. Each layer should be replaceable without affecting the others. Module dependency direction: `routes → services → db`, `routes → viewmodels → ui`. Detailed rules in `docs/internal/coding-standards.md`.

- **Routes are thin adapters**: a route handler should only parse/validate the request, call one or more service methods, and format the response. Multi-service orchestration (e.g. "delete a post and clean up its media files") belongs in the service layer, not in routes. **Litmus test**: if two routes need the same sequence of service calls, that sequence must be extracted into a service method. Cross-cutting concerns like storage file cleanup are passed to services via optional dependency parameters (e.g. `storage?: StorageDriver | null`) rather than being handled in routes.

- **Type safety as communication**: TypeScript strict mode with no `any` and fully typed exports prevents silent contract drift between layers. When a service return type changes, the compiler should catch every consumer.
- **Normalize interpolated copy inputs before rendering**: when user-facing copy interpolates labels, hosts, dates, counts, or settings, normalize the value first. Empty or whitespace-only strings must fall back explicitly before they reach the UI. Do not rely on truthiness fallbacks for numeric or boolean values because `0` and `false` are often valid data. In translated copy, never bake runtime values into the `message` string itself; use placeholders plus `values` so extraction, translation, and fallback behavior stay correct.
- **Hosted integrations stay neutral in core**: when `jant-core` integrates with a hosted control plane, keep the runtime and copy provider-neutral. Brand names belong in the control plane, not in core. Core may show a configured provider label, and when no label is configured it should fall back to the provider host/domain instead of hardcoding product branding.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jant-me/jant](https://github.com/jant-me/jant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
