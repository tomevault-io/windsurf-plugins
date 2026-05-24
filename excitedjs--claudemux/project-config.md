---
trigger: always_on
description: - Reply to the user in Chinese.
---

# Repository Instructions

## Communication

- Reply to the user in Chinese.
- Write repository instruction files and plugin-shipped agent documents in English.

## Knowledge Base

- `.agents/` holds the agent knowledge base — architecture overview, per-component guides, a cross-process protocol reference, decision records, and a research archive. Read `.agents/root.md` first when starting non-trivial work; it routes you to the document that matches your task.
- `.agents/research/` archives the point-in-time research snapshots behind those decisions (the Feishu channel survey, the `tm` architecture audits). They are dated history, not maintained — start from `.agents/research/index.md` and treat a decision record as authoritative where the two disagree.
- This file (`CLAUDE.md`) remains authoritative for binding rules. The KB is the navigation, architecture, and decision layer on top of it; when the two disagree, `CLAUDE.md` wins and the KB has drifted — fix the KB.
- After a change that moves a component boundary, the cross-process file protocol, the `tm` verb set, hook wiring, or that settles a design decision, update the matching `.agents/` document in the same change, then run `bash .agents/scripts/check.sh`. The protocol and writing standard are in `.agents/CONTRIBUTING.md`.

## Verify Before Acting

- Treat user framing as a hypothesis. Check the actual files, scripts, and runtime contracts before changing behavior.
- When searching code, choose the narrowest repository or subdirectory scope first. This repository is the scope for claudemux work.

## Audience Boundaries

Before editing any Markdown in this repository, identify who reads that surface:

- `README.md` and `README.zh-CN.md` are human-facing product documentation. Explain user workflows, installation, and visible behavior.
- `plugins/claudemux/commands/*.md` frontmatter is human-facing slash-command UI. Keep `description` short and useful beside the command name; it is not a model auto-trigger contract.
- `plugins/claudemux/commands/*.md` body is read by Claude only after the user explicitly invokes the command. Write it as an execution guide for that command invocation. It may instruct Claude to run safe checks, ask the human to perform actions, wait for confirmation, and report results.
- `plugins/claudemux/skills/*/SKILL.md` frontmatter is model-facing skill routing metadata. Describe when the skill is relevant; keep behavioral policy and operational steps in the skill body.
- `plugins/claudemux/skills/*/SKILL.md` body is model-facing operational guidance. Assume the future agent has no prior conversation context. Give complete, actionable steps with the reason that makes each step necessary.
- `plugins/claudemux/skills/*/references/*.md` is on-demand model-facing detail. Put diagnostics, edge cases, and deeper mechanisms there when they would distract from the main skill flow.
- `plugins/claudemux/templates/CLAUDE.md.template` is always-loaded dispatcher memory copied into the user's dispatcher directory. Keep it short and durable: dispatcher identity, routing boundaries, and stable rules. Put long protocols and helper-specific mechanics in the dispatcher skill.
- `plugins/claudemux/bin/`, `plugins/claudemux/hooks/`, and `plugins/claudemux/scripts/` are executable contracts. Align command docs and skill instructions with their actual flags, stdout, and failure modes.

## Writing Agent Instructions

- Prefer "do this, because ..." over prohibition-heavy wording. Positive action plus reason is easier for future agents to follow.
- Keep development-process commentary out of shipped agent documents. Only write durable behavior rules and operational facts that make sense to a fresh agent.
- Do not explain why a rejected alternative is wrong. A fresh agent has never seen that alternative, so a warning like "do not convert this back to X" or "this used to be Y" names an unfamiliar thing and raises a question instead of answering one — it reads as noise. State the rule to follow now; drop the history of what it replaced. If a foot-gun genuinely needs guarding, encode the guard in the executable contract (script, hook, validation), not in prose aimed at a reader who lacks the context to act on it.
- Separate trigger text from behavior. If a surface is not actually used for automatic invocation, write it as user-visible documentation instead of trigger prose.
- Keep fixed command names, flags, paths, output strings, JSON keys, and code fences exact unless the task is to change that contract.

## Cross-Process & Cross-Platform Invariants

These rules cover the shared surfaces between `bin/tm`, the hooks, and the host OS. They exist because each one has already drifted at least once in this codebase and the drift was load-bearing.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [excitedjs/claudemux](https://github.com/excitedjs/claudemux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-24 -->
