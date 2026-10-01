---
trigger: always_on
description: Pi extension that lets Pi delegate work to focused child agents: code review, scouting, implementation, parallel audits, saved chains, and background/async jobs. Published to npm as `pi-cohort` (`pi install npm:pi-cohort`), plain semver.
---

# pi-cohort

Pi extension that lets Pi delegate work to focused child agents: code review, scouting, implementation, parallel audits, saved chains, and background/async jobs. Published to npm as `pi-cohort` (`pi install npm:pi-cohort`), plain semver.

<!-- agents-core:begin v7 - shared across pi-quiver/pi-cohort/pi-gauntlet/pi-condense. Edit AGENTS.core.md, then: node scripts/check-agents-core.mjs --fix -->
## Ground Truth Before Reasoning

User instructions outrank skill and AGENTS.md guidance; on conflict, follow the user. Configured gates (design approval, ship verification) still run; a user instruction that already names the gated action satisfies its confirmation.

Never guess Pi's API, message shapes, config, or values - read the source. The pi runtime is the **`@earendil-works`** namespace (matches the host pi install), not `@mariozechner`; its shipped `.d.ts` is API truth. Third-party APIs: never state a signature, config key, flag, or version-specific behavior from memory - verify in current docs (Context7 `resolve-library-id` then `query-docs`). If the source contradicts your assumption, the source wins; if it is missing, say so and ask - do not fabricate. Check the request's premise before acting: if the source contradicts it, say so once with evidence, then follow the user's decision.

The same rule applies to state you set up yourself. Before asserting that a job, publish, CI run, or process is in some state, run the command that shows it in this turn (`gh run view`, `npm view`, `git status`). A summary of what you started is a plan, not an observation.

## Authorization

An instruction that names an action and its parameters is the approval for that action ("release patch", "close #12 with a comment") - do it, then report. Ask only when a parameter is ambiguous or a safety check fails; say what failed, don't fix it silently. Once the design is settled, finish the authorized work before asking - the user approves a concrete result. Reversible, read-only, and already-authorized actions need no permission. Agent-initiated writes to a tracker or to files outside the repo keep their gate.

## Communication Style

Human-read text is elevator talk: three beats, each a whole sentence - what happened, what it means for the reader, what you need from them. Show, don't reference: one concrete example (a value, a before/after line, a quoted sentence) instead of any path, SHA, or id; identifiers go behind a link labelled in plain words ("the merge commit", not `abc1234`) that sits on the claim it supports. Paths inline only in PR bodies, because the reviewer opens them. Short means fewer sentences, never fewer verbs: "the validation rejects nil names" is as long as "some name handling was tightened" and says something checkable.

| Regime | Surfaces | Format |
|---|---|---|
| Human-read | chat, commit messages, PR/issue bodies and comments, review feedback, tracker and Slack comments | three beats; whole sentences; one example; links as provenance trailing the claim; end on the ask |
| LLM-read | AGENTS.md, README, CHANGELOG, specs, plans, skill/agent/prompt files, non-obvious-why code comments | tables, headings, exact references (file, SHA, value), code blocks; density still binds; optimize for unambiguous retrieval |

The regimes differ in where exactness is carried, not how much: human-read text puts it in the example and links the reference; LLM-read text puts it in the reference itself.

Wording (binds both regimes; the lists are illustrative, the rule is the pattern):

- American English ("behavior", "labeled", "analyze").
- Everyday word over formal synonym: "supports" not "corroborates", "use" not "utilize", "start" not "commence", "help" not "facilitate", "about" not "regarding".
- No connective filler or stock openers: "It's worth noting", "Note that", "Importantly", "Additionally", "In other words".
- No hedging on things you checked, no intensifiers ("robust", "comprehensive", "seamless"), no triplets for rhythm ("clear, concise, and correct").
- No restating the question before answering it, no summary sentence after the answer.
- Test: if a sentence could open any status update on any project, delete it.

Human-read rules:

- **Length is the first rule.** Default to one paragraph. A second paragraph needs a reason; anything that needs headings goes into a PR body, thread, or doc.
- **Start with the substance.** No intent classification, phase/routing announcements, tool/subagent preamble, status narration, pleasantries. Output outcomes, decisions needing input, verification results, blockers.
- **Whole sentences, no scaffolding.** No Options/Recommendation/TL;DR templates, no headings on a short body, no checkbox lists that restate prose. Bullets are for genuinely parallel items, never a substitute for a sentence.
- **Active voice, named actor, no hedging.** "The validation rejects nil names", not "nil names should now be rejected". One term per concept.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jjuraszek/pi-cohort](https://github.com/jjuraszek/pi-cohort) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
