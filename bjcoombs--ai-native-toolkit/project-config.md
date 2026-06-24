---
trigger: always_on
description: In-repo contract for agents editing `ai-native-toolkit`. Repo-specific rules only - global conventions (voice, workflow, worktrees) live in the user's `~/.claude/CLAUDE.md` and aren't repeated here.
---

# CLAUDE.md

In-repo contract for agents editing `ai-native-toolkit`. Repo-specific rules only - global conventions (voice, workflow, worktrees) live in the user's `~/.claude/CLAUDE.md` and aren't repeated here.

## Scope of this repo

Source for the `ai-native-toolkit` Claude Code plugin. Portable skills (`/assess`, `/huddle`, `/deslop`, `/ghsync`, `/skill-forge`, `/semantic-compress`), portable framework commands (`/6hats`, `/understand`), personal workflow commands that are opt-in (`/tm`, `/issues`, `/fix-pr`, `/fix-develop`), and team-orchestration library skills (`marathon`, `pr-review-merge`, `ab-equivalence`) that are composed by the workflow commands and portable skills rather than invoked standalone.

The deliverable is markdown: agents, commands, skills. It also ships a Python deterministic core under `skills/assess/scripts/` (plus `lib/`) and a standalone-skill build pipeline under `scripts/`. There is no application runtime, but there are pytest suites (`skills/assess/`, `scripts/`), a ruff + mypy lint gate, and a standalone-ZIP build step - all enforced in CI.

## North star

Everything here serves one goal: make an AI contributor feel like an engineer who has been in the org eighteen months, not a brand-new hire. The difference is *externalized context*, not capability. An AI is structurally always the new hire - each session starts with an empty head and one narrow context window - so the codebase has to supply the tenure: a navigable map, load-bearing contracts made explicit, complexity made *locally* legible so the relevant slice fits one keyhole.

The safety half is non-negotiable: the goal is **legibility you can trust, not omniscience you can't.** An agent fluent about code nobody can verify is the dangerous failure, not the win. Answers must stay anchored to code a human can spot-check.

The other half is the same ethic pointed at the write side: when a contributor makes a mistake, ask "what made it possible, and what would make it impossible next time?" - not "who's to blame". Guardrails (linters, architecture tests, CI gates, coverage, review automation - Layers 3-7) aren't a leash on the AI; they protect the contributor from costly mistakes by design, the way an org protects a human engineer with RBAC and staged environments rather than hope. This is **correctness by construction** / poka-yoke: make the wrong action hard and the right action the path of least resistance.

The write-side mistakes worth guarding against are not hypothetical - they are the known tendencies of an AI contributor, observed across models and worth naming so every guardrail traces to one:

- **Accretion.** An agent does what is asked, and what is asked is feature after feature. Nothing in that loop ever asks for a refactor, so files only grow. Absent a consciously requested restructuring, size and complexity ratchet monotonically upward.
- **Unactioned intent.** An agent records promises it never returns to keep: TODO / FIXME / "deprecated, use X" / "remove after migration" comments. These are *promissory markers* - self-descriptions of the code's future with no pressure to come true. Aged and ignored, they are a lying map of intent, exactly as a stale doc is a lying map of behaviour.
- **Guardrail erosion.** Under pressure to make red go green, an agent loosens the check instead of fixing the root - a suppression here, a skipped test there, a widened threshold - hollowing out the very layers meant to protect it while the scaffolding still reads as Present.

All three are the same defect: a self-description (the file's shape, the comment's promise, the gate's verdict) under no pressure to stay true. The toolkit's job is to convert each tendency into a deterministic signal (the marker aged by the edits it survived, the file that only ever grows, the suppression count that climbs) and a ratchet that makes the honest action the cheap one.

**Use this as the feature test.** Judge any change to a skill or report by: does it help a fresh agent (the map), keep its answers verifiable (trust), *and* make the wrong action hard to take (guardrails - protecting the contributor, not policing it)? When adding a signal or rule, name the contributor tendency it compensates for - a guardrail that doesn't trace to a failure mode is decoration. Prefer honest-degrade over impressive-but-wrong, local comprehension over global, and "ground the claim in the file" over a fluent narrative.

## Versioning

The plugin follows [semver](https://semver.org). Version lives in `.claude-plugin/plugin.json` under `.version`.

| Bump | When |
|------|------|
| MAJOR | Breaking change to a skill or command (rename, removed flag, behaviour change users would need to adapt to) |
| MINOR | New skill, new command, new feature on an existing skill (`--include-artifacts` was a MINOR) |
| PATCH | Bug fix, doc-only change, refactor with no user-visible effect |

**Bump in the same PR as the change.** Claude Code's `/plugin update` is version-based - users see "already at latest" and miss changes if the version doesn't move. A trailing version-bump PR is a fix-up, not the pattern to follow.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bjcoombs/ai-native-toolkit](https://github.com/bjcoombs/ai-native-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
