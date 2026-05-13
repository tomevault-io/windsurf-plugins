---
trigger: always_on
description: H·AI·K·U = Human + AI Knowledge Unification — a universal lifecycle framework for structured AI-assisted work.
---

# H·AI·K·U Project

H·AI·K·U = Human + AI Knowledge Unification — a universal lifecycle framework for structured AI-assisted work.

Three-component project: **plugin** (Claude Code plugin), **paper** (methodology spec), **website** (Next.js 15 static site).

- Paper is the source of truth for methodology concepts
- Plugin is the source of truth for implementation (orchestrator, MCP tools, hooks, runtime behavior)
- **`plugin/studios/ARCHITECTURE.md` is the structural reference for studio/stage/unit/hat/feedback boundaries within the plugin** — the rules and contracts that apply across studios, distinct from any single implementation file. Read it before structural changes; conflict resolution between this doc and the plugin code is described in the doc's intro.
- Website presents all of the above to users

## Sync Discipline (CRITICAL)

When modifying any component, check if other components need corresponding updates:

| Change Type | Paper | Plugin | Website |
|---|---|---|---|
| New prompt | Mention in relevant section | Add handler in `prompts/*.ts` | Update docs if user-facing |
| New studio | Document in Profiles section | Primary | Update docs |
| New stage | Document in relevant profile | Primary | Update docs |
| New hat (in stage) | Document in relevant profile | Add `hats/{hat}.md` file in stage directory | Update docs if user-facing |
| New review agent (in stage) | Document in Quality Enforcement | Add `review-agents/{agent}.md` file in stage directory | Update docs if user-facing |
| New fix hat (in stage) | Mention in Fix Loop section | Add `hats/{hat}.md` file in stage directory + reference in `fix_hats:` on STAGE.md | Update docs if user-facing |
| New studio-level review agent | Mention in Intent-Completion Review section | Add `review-agents/{agent}.md` file in studio directory (NOT per-stage) | Update docs if user-facing |
| New studio-level fix hat | Mention in Intent-Completion Review section | Add `fix-hats/{hat}.md` file in studio directory (NOT per-stage) | Update docs if user-facing |
| New phase override (in stage) | Mention in Stages section if needed | Add `phases/{PHASE}.md` file in stage directory | Update docs if user-facing |
| New operation template | Document in Operation phase | Add `operations/{op}.md` file in studio directory | Update docs if user-facing |
| New reflection dimension | Document in Reflection phase | Add `reflections/{dim}.md` file in studio directory | Update docs if user-facing |
| New lifecycle phase | Document as new section | Implement | Update docs |
| Terminology change | Update all references | Update all references | Update all references |
| New principle | Document in Principles section | Implement if applicable | Update if referenced |
| Concept refinement | Update definition | Update implementation | Update docs |
| Persistence change | N/A (environment-detected) | Update state-tools.ts isGitRepo | Update docs if user-facing |
| New harness support | N/A | Add entry to HARNESS_REGISTRY in `harness.ts`, update rewriting rules in `harness-instructions.ts` | Update docs if user-facing |

## Key File Locations

- Paper: `website/content/papers/haiku-method.md`
- Plugin metadata: `plugin/.claude-plugin/plugin.json`
- Plugin prompts: `packages/haiku/src/prompts/*.ts` (MCP prompt handlers — all behavior lives here)
- Plugin studios: `plugin/studios/*/STUDIO.md`
- Plugin stages: `plugin/studios/*/stages/*/STAGE.md`
- Plugin hats: `plugin/studios/*/stages/*/hats/*.md`
- Plugin review agents: `plugin/studios/*/stages/*/review-agents/*.md`
- Plugin phase overrides: `plugin/studios/*/stages/*/phases/*.md`
- Plugin operations: `plugin/studios/*/operations/*.md`
- Plugin reflections: `plugin/studios/*/reflections/*.md`
- Plugin studio-level review agents (intent-completion review): `plugin/studios/*/review-agents/*.md`
- Plugin studio-level fix hats (intent-completion fix loop): `plugin/studios/*/fix-hats/*.md`
- Plugin intent templates: `plugin/studios/*/templates/*.md`
- Plugin hooks: `plugin/hooks/*.sh` + `plugin/.claude-plugin/hooks.json`
- Plugin libraries: `plugin/lib/*.sh`
- Plugin orchestration: `plugin/lib/orchestrator.sh`, `plugin/lib/stage.sh`, `plugin/lib/studio.sh`
- Plugin environment detection: `packages/haiku/src/state-tools.ts` (isGitRepo)
- Plugin harness support: `packages/haiku/src/harness.ts` (capability registry), `packages/haiku/src/harness-instructions.ts` (instruction adaptation)
- Plugin providers: `plugin/providers/*.md` (bidirectional translation instructions) + `plugin/schemas/providers/*.json`
- Website docs: `website/content/docs/`
- Infrastructure: `deploy/terraform/`
- Changelog: `CHANGELOG.md` (Keep a Changelog format)

## Concept-to-Implementation Mapping

| Concept | Paper Section | Plugin Implementation | Key Files |
|---|---|---|---|
| Intent | Elaboration phase | `.haiku/intents/{slug}/intent.md` | prompts/core.ts |
| Unit | Elaboration phase | `.haiku/intents/{slug}/stages/{stage}/units/unit-NN-*.md` | prompts/core.ts |
| Bolt | Execution phase | `iteration` field in iteration.json | orchestrator.ts |
| Studio | Profiles section | `plugin/studios/{name}/STUDIO.md` | studio.sh |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gigsmart/haiku-method](https://github.com/gigsmart/haiku-method) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
