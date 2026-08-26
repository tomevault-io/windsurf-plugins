---
trigger: always_on
description: MDDOAI (Model-Driven DevOps AI) generates CI/CD pipeline configs from software architecture models, without requiring MDE expertise, via two tracks:
---

# CLAUDE.md — repo root

## What This Project Is

MDDOAI (Model-Driven DevOps AI) generates CI/CD pipeline configs from software architecture models, without requiring MDE expertise, via two tracks:

- **The MDE engine** (repo root): a Java/Eclipse EMF/ATL/Acceleo transformation chain, `SWArch → PIM → PSM → YAML`. See [README.md](README.md) for build/usage.
- **The AI product** (`ai/`): a chat-based agent system built on top of the same chain. See [ai/README.md](ai/README.md) and [ai/CLAUDE.md](ai/CLAUDE.md).

## Repo Structure

- `main/`, `meta_models/`, `code_generation/`, `designs/`, `feature/`, `update_site/` — the Java/Eclipse MDE engine and its transformation artifacts.
- `ai/` — the AI product: multiple services (chat UI, backend API, and supporting agents) built on top of the transformation chain. Mostly separate from the Java/Eclipse engine, with one narrow, explicitly documented exception. See `ai/CLAUDE.md` for the documented folder boundaries and the exception's exact scope.
- `mddoai-design-system/` — the on-brand component library and Claude Design skill (`/mddoai-design`). Read `mddoai-design-system/project/SKILL.md` before doing UI work.
- `docs/` — misc project docs.
- `pipeline_tests/`, `install_necessary_packages/`, `viewpointrepresentations/` — supporting material for the MDE engine; match a new file's placement to the existing sibling closest to its purpose.
- `logo/` and top-level project docs — general project branding and documentation, shared across the MDE engine and the AI product.
- If a directory doesn't appear anywhere in this list, that's a gap in this section to flag and fix, not a signal that a file placed there is automatically wrong.

## Agents in `.github/agents/`

These are GitHub Copilot agent definitions, run from VSCode's Copilot Chat agent picker, not tools Claude Code can invoke directly. When a task matches one of their purposes, open the `.agent.md` file and carry out its steps directly instead of trying to call it as a tool. `lint-reviewer`, `oop-reviewer`, and `coverage-reviewer` overlap with the independent-review need described under [Review](#review) below for Java changes specifically; the Review section's `/code-review`/subagent mechanism is what Claude Code itself can actually run.

- `pr-logic-reviewer` — review a PR's actual logic/diff (`pr=<number>`)
- `pr-description-generator` — write a PR description from the current branch's diff against main
- `coverage-reviewer` — run the Gradle suite, report JaCoCo coverage gaps by class
- `lint-reviewer` — Java formatting/lint issues (naming, method length, magic numbers, nesting)
- `oop-reviewer` — Java OOP design quality (SOLID, code smells, encapsulation)
- `test-fixtures-updater` — after a transformation change, re-run swarch2pim/input1.swarch and update expected + downstream fixtures

## Git Workflow — read before pushing or merging

- **Never force-push `main`, full stop.**
- **Never rebase or force-push any other branch that's already been pushed to origin.** Check first with `git ls-remote --heads origin <branch>`, if that returns nothing, the branch is local-only and rebasing is safe. If it returns a ref, merge instead. This project has already had one real incident where a force-push wiped pushed commits before a stripped-down version got merged into `main`, recovering required a separate revert PR. Don't repeat it.
- **Merge `main` into your feature branch, not the other way around.** Never merge a feature branch directly into `main` outside of a reviewed PR.
- **Do not commit unless explicitly asked.** Wait for a direct instruction to commit.
- **If you do commit, keep the message to 5-6 words, one line.** No large multi-paragraph bodies.
- **Do not add a co-author line to any commit.**
- **Run `git status` before any destructive command** (`checkout --`, `restore`, `reset --hard`, `clean`) on a path that might have uncommitted work.
- **Confirm before merging PRs**, even on your own branches, unless explicitly told to proceed autonomously.
- **Name new branches `<type>/<short-description>`** (`feature/`, `fix/`, `docs/`, `refactor/`). Some existing branches (the `feature/*` and `docs/*` ones) already use this pattern; use it going forward.

## Engineering Standards

A change is **non-trivial** if it touches more than one small file, changes a public interface or contract, changes behavior a test could observe, or touches shared/production config. A one-line fix or a pure rename is trivial. A decision meets the same bar if the change it leads to would meet it. This definition is what "nontrivial"/"non-trivial" means everywhere it's used below.

### Before you build

- Before writing new code for a capability, check whether a well maintained library, an established design, or an existing pattern already in this repo solves it (grep this repo for similar functionality; check `build.gradle` or the relevant `requirements.txt`/`package.json` for an already-available library before adding a new dependency). Write custom code only when nothing suitable exists, or there's a specific, stated reason existing options don't fit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modeldrivendevopsai/mddoai](https://github.com/modeldrivendevopsai/mddoai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
