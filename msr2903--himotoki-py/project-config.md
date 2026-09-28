---
trigger: always_on
description: This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.
---

# Agent Instructions

This project uses **bd** (beads) for issue tracking. Run `bd onboard` to get started.

## Project Overview

**Himotoki** is a Japanese text segmentation library that breaks sentences into words with readings.

**Key directories:**
- `himotoki/` - Core library (segment.py, splits.py, lookup.py, suffixes.py, conjugation_hints.py)
- `scripts/` - Evaluation tools (llm_eval.py, check_segments.py, llm_report.py)
- `tests/` - Test suite
- `data/` - Dictionary data, skip lists, lock files
- `output/` - Evaluation results (llm_results.json, llm_report.html)

**Current task:** Improve segmentation accuracy by fixing failures in LLM evaluation.

## Agent Onboarding

When starting a new session, run these commands:

```bash
bd onboard                                    # Learn beads issue tracking
python scripts/llm_eval.py --triage-status    # See failed entries & progress
bd ready                                      # Find available work
git log --oneline -5                          # Recent changes context
```

**First-time setup:**
```bash
source .venv/bin/activate                     # Activate Python environment
pip install -e .                              # Install himotoki in dev mode
pytest tests/ -x --tb=short                   # Verify tests pass
```

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --status in_progress  # Claim work
bd close <id>         # Complete work
bd sync               # Sync with git
```

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

## Codebase Search

Use **chunkhound MCP** for semantic and regex codebase searches. Returns contextualized results with file paths and line numbers.

## Maximizing Agent Autonomy

Agents should work efficiently but **always confirm bug decisions with the user**. Follow these principles:

### Autonomous Decision Framework

**DECIDE YOURSELF** (don't ask):
- Implementation approach when issue description is clear
- Commit message wording
- Which files to change based on root cause analysis
- Test verification strategy

**ALWAYS ASK USER** (use `ask_questions` tool):
- Whether to fix or skip ANY bug - always get confirmation
- Before creating any beads issue
- Before skipping any entry

### Skip/Fix Pattern Reference

Use these patterns to **suggest** actions to the user, but always confirm:

**Common Skip Patterns** (suggest skip, confirm with user):

| Pattern | Suggested Reason |
|---------|------------------|
| Proper noun reading | `Proper noun - multiple valid readings` |
| Counter ambiguity | `Counter expression - valid alternative` |
| Stylistic particle | `Stylistic choice - both valid` |
| Archaic reading | `Archaic reading - dictionary limitation` |
| Compound boundary | `Compound boundary - subjective split` |

**Common Fix Patterns** (suggest fix approach, confirm with user):

| Pattern | Suggested Action |
|---------|------------------|
| Missing suffix | Add suffix to `himotoki/suffixes.py` |
| Wrong POS tag | Update POS mapping in `himotoki/lookup.py` |
| Missing dictionary entry | Add to custom dictionary |
| Conjugation error | Fix in `himotoki/conjugation_hints.py` |

### Error Recovery

**Git conflicts:**
```bash
git stash && git pull --rebase && git stash pop
# If conflict persists, resolve manually then continue
```

**Test failures after fix:**
1. Check if failure is related to your change
2. If unrelated, note it and continue
3. If related, revert and try different approach

**Push rejected:**
```bash
git pull --rebase && git push  # Retry
# If still fails, check for large files (>100MB)
```

**Chunkhound lock error:**
```bash
kill $(lsof -t .chunkhound/db.wal) 2>/dev/null
rm -f .chunkhound/db.wal
```

### Batch Processing

Process in batches to maximize throughput:

**Triage Agent:**
- Analyze 5-10 entries before asking user for batch confirmation
- Group similar failures together
- Present: "Found 3 suffix issues, 2 reading issues. Auto-fix suffixes, ask about readings?"

**Fix Agent:**
- Implement up to 3 related fixes before committing
- Run tests once after batch, not after each fix
- Single commit for related fixes with detailed message

### Progress Checkpoints

Save state frequently to avoid losing work:

```bash
# After every 3-5 items processed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [msr2903/himotoki-py](https://github.com/msr2903/himotoki-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
