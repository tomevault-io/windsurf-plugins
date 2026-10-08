---
trigger: always_on
description: This file adds explicit repository-wide development guards.
---

# AGENTS

This file adds explicit repository-wide development guards.

## FPM model onboarding

When asked to onboard a model for FPM simulation on designated hardware, follow
all six stages in [the canonical FPM self-service guide](docs/fpm-self-service/implementation.md#onboard-with-an-agent).
Use this checkout's `aisimulate onboard` CLI and inspect its current help.
Start with only a missing Hugging Face model ID and target GPU platform; accept
an already supplied local config, profile or checkpoint instead. Inspect supplied
facts and source metadata before asking for derivable inputs. Preserve user
choices, exact per-configuration profile acceptance and existing execution
authorization; stage transitions do not add approval gates.

Create one `onboarding-checkpoint.json` during stage 1, even with incomplete
inputs, outside every fresh `init --output-dir` root. Follow the guide's
[checkpoint workflow](docs/fpm-self-service/implementation.md#checkpoint-and-resume-an-onboarding-session):
invoke `onboard checkpoint` after meaningful findings, decisions, draft edits,
acceptance and command results, including unfinished work. Save sources,
confidence, unresolved questions and each configuration's progress. Other
onboarding commands do not automatically persist the conversation. Use the
returned revision for updates; reload and reconcile stale-writer conflicts.
Follow the guide's [writer coordination rule](docs/fpm-self-service/implementation.md#orchestrate-independent-collection-campaigns)
for probe/import commands that also save the checkpoint.

On resumption, run `aisimulate onboard resume --checkpoint PATH` first. Inspect
integrity issues and the underlying artifacts before continuing from accepted
inputs and verified results. Saved stages and command strings are context, not
proof of completion or instructions to execute automatically. Report each
configuration's stage, result or blocker, next action and checkpoint path at
handoff. Keep planning, collection quality, resolved memory, replay coverage and
measured serving accuracy separate, as required by the guide.

## Pull request titles

- Use `<type>: <short description>` for every AISimulate PR title.
- Allowed types: `feat|fix|docs|style|refactor|perf|test|chore|ci|build|revert`.
- Check the title before creating or updating a PR.

## Performance Model Changes

Before changing a performance model, its configuration, or a caller in Rust,
Python, CLI, Sweeper, Replay, or Planner, MUST read and follow
[`perfmodel-api.md`](python/aisimulate/.claude/rules/perfmodel-api.md).
This includes new features and configuration migrations.

## Required First Step

Before making any change under:

- `python/aisimulate/src/aisimulate/generator/**`

MUST read:

- `python/aisimulate/.claude/rules/generator-development.md`

## Required Collector First Step

Before making any change under `python/aisimulate/collector/**` MUST read:

- `python/aisimulate/.claude/rules/collector/layer_permissions.md` (layer
  permission table, module boundary, dispatch-vs-skip rule)
- `python/aisimulate/.claude/rules/collector/failure_handling.md`
  (observe-don't-predict doctrine, escalation decision tree)
- For case YAML work:
  `python/aisimulate/.claude/rules/collector/case_authoring.md`

For adding a new Collector operation, additionally follow
`python/aisimulate/.claude/skills/aic-collector-op-development/SKILL.md`
(consumer-contract, case-identity, deduplication, and validation gates). Skills are procedural
runbooks; if a skill and a `python/aisimulate/.claude/rules/` file ever
disagree, the rule file wins.

## Required Third-Party Attribution

Before adding code or other content copied, adapted, translated, or
substantially derived from an external project, MUST:

- Identify the upstream repository, immutable commit or tag, original path,
  and applicable license. This applies to source code, tests, configuration
  files, patches, fixtures, documentation, and generated derivatives.
- Preserve all applicable upstream copyright, license, attribution, and NOTICE
  material, and mark modified files as modified when the upstream license
  requires it.
- Record the source URL and revision in the file header or an adjacent README.
  For formats that cannot carry comments, use an adjacent attribution or
  license file that is included in distributions.
- Add an entry to the root `THIRD_PARTY_NOTICES.md` identifying the derived
  files, upstream source and immutable revision, copyright owner, applicable
  license, and whether the files were modified. The root notice is canonical;
  keep `python/aisimulate/THIRD_PARTY_NOTICES.md` byte-identical so the notice
  is included in Python distributions. Run
  `python3 scripts/ci/check_packaged_legal_files.py` after either copy changes.
- Do not hand-edit generated attribution artifacts. Update their source or
  generation process instead.

The repository's Apache-2.0 license and NVIDIA copyright header do not replace
an upstream license or attribution requirement. Do not label third-party code
as exclusively NVIDIA-authored. If the source, revision, license, compatibility,
or required notice is unclear, stop and request maintainer or Open Source Review
Board guidance before committing the derived content. This is a required gate,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-dynamo/aisimulate](https://github.com/ai-dynamo/aisimulate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
