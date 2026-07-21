---
trigger: always_on
description: `typia` is a TypeScript transformer and runtime library that turns pure TypeScript types into inline validators, serializers, schemas, decoders, and random-data generators. It ships the `typia` CLI plus the `typia`, `@typia/interface`, `@typia/utils`, `@typia/langchain`, `@typia/mcp`, and `@typia/vercel` packages.
---

# AGENTS.md

`typia` is a TypeScript transformer and runtime library that turns pure TypeScript types into inline validators, serializers, schemas, decoders, and random-data generators. It ships the `typia` CLI plus the `typia`, `@typia/interface`, `@typia/utils`, `@typia/langchain`, `@typia/mcp`, and `@typia/vercel` packages.

## Attitude

Follow the literal request; it is the contract, not a hint at what the user "really" wants.

- **Scope is the user's to widen.** Reinterpret the goal, weigh alternatives, or expand the task only on an explicit hand-off ("figure it out", "you decide"). Take a confident, specific ask as given.
- **Fidelity binds the goal, not the effort.** Within that goal, act with full initiative: do the substeps it needs, verify your work, surface what you notice. Literal scope is no excuse for passive execution.
- **Match the user's language.** Communicate in English when the user writes in English and in Korean when the user writes in Korean. Switch when the user switches, unless they explicitly request another language.
- **Choose the principled course.** Decide from evidence, correctness, product boundaries, and the durable consequence. Time, difficulty, and consequence surface are reasons to investigate and validate more carefully, never reasons to settle for a shortcut, workaround, or weaker standard.
- **Evidence precedes correction.** Treat issue reports, review proposals, and claims that something is wrong or missing as hypotheses. Verify the real code path, tests, generated artifacts, upstream ownership, and history before accepting the premise or changing behavior.
- **Trace the consequence surface.** A named file or failing case is the starting point, not the investigation boundary. Follow the same cause through downstream consumers, side effects, state transitions, platforms, and boundary cases, then address the whole verified class of failure within the requested goal.
- **Default over ask.** On an ambiguous detail, pick the sensible default and say what you chose; reserve questions for forks only the user can settle.
- **Fill the available agent pool when parallel work is independent.** Use up to ten concurrent agents in total, including the lead, when the runtime provides that capacity; keep at most nine child agents active so the lead retains one coordination slot.

## Skills

Durable project conventions and workflows live under `.agents/skills/`. Read the linked skill when its topic applies; each skill indexes its own conditionally needed topic documents.

### Project Outline

What typia is, the package family, the JS-descriptor / Go-plugin boundary, the workspace layout, product boundaries, and canonical commands, `.agents/skills/project/SKILL.md`.

### Development

Work rules, testing, validation, consequence analysis, and change integrity, `.agents/skills/development/SKILL.md`. Read before writing or modifying code.

### Documentation

README and website guide authoring rules, `.agents/skills/documentation/SKILL.md`. Read before writing or modifying documentation.

### Issue Campaigns

Repository-wide issue discovery, lead-verified issue writing, CI-suspended DAG-batched implementation, and cleanup, `.agents/skills/issue-campaign/SKILL.md`. Read when the user asks for a broad audit, many issue candidates, or an issue-to-implementation campaign; do not use it for one already-defined issue.

### Review

Solo review, team Review Cycle, research review, and exhaustive issue-discovery rounds, `.agents/skills/review/SKILL.md`. Every agent inspects the whole declared surface independently, and rounds continue until a complete pass produces no sound improvement or meaningful issue candidate. Self-Review and any unqualified review request are always solo.

### Discussion

Structured multi-agent topic discussion with persistent research notes and transcripts, `.agents/skills/discussion/SKILL.md`. Read only when the user explicitly asks for a discussion; review and issue discovery do not imply discussion.

### Pull Request Submission

Branch, commit, pull request, check, and merge flow, `.agents/skills/pull-request/SKILL.md`. Read when the user explicitly asks to open, submit, update, or merge a pull request, or when a standing autonomous mandate authorizes end-to-end delivery; never open, push, or propose one on unprompted initiative.

### Benchmark

Benchmark runners, fixture integrity, measurement integrity, result archives, and publication, `.agents/skills/benchmark/SKILL.md`. Read before running, modifying, or publishing benchmark results.

## Maintenance

### Writing style

AGENTS.md and SKILL.md files are read by humans as well as agents.

- **Optimize for comprehension, not minimum length.** A shorter document that forces the reader to infer prerequisites, reasons, exceptions, or stop conditions is not concise. Add the context needed to execute correctly.
- **Remove repetition, not substance.** State a rule once at its owner and link to it elsewhere. Keep the rationale when it prevents a plausible mistake.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samchon/typia](https://github.com/samchon/typia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
