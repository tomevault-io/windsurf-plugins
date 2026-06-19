---
trigger: always_on
description: |
---

<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## Pre-check

Check the SessionStart hook output in this conversation context for `ADEEL_AUTO_UPDATE=`.
If it says `ADEEL_AUTO_UPDATE=false`, use AskUserQuestion to ask:
"Auto-updates are off. Run /adeel-update to enable?" If yes, invoke
`/adeel-update`. If `ADEEL_AUTO_UPDATE=true` or not found, proceed directly
without mentioning it.

## Preamble (run first)

```bash
mkdir -p $HOME/.adeel/sessions
touch $HOME/.adeel/sessions/"$PPID"
_SESSIONS=$(find $HOME/.adeel/sessions -mmin -120 -type f 2>/dev/null | wc -l | tr -d ' ')
find $HOME/.adeel/sessions -mmin +120 -type f -delete 2>/dev/null || true
_CONTRIB=$(${CLAUDE_PLUGIN_ROOT}/bin/adeel-config get adeel_contributor 2>/dev/null || true)
_PROACTIVE=$(${CLAUDE_PLUGIN_ROOT}/bin/adeel-config get proactive 2>/dev/null || echo "true")
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
echo "PROACTIVE: $_PROACTIVE"
source <(${CLAUDE_PLUGIN_ROOT}/bin/adeel-repo-mode 2>/dev/null) || true
REPO_MODE=${REPO_MODE:-unknown}
echo "REPO_MODE: $REPO_MODE"
echo "LAKE_INTRO: $_LAKE_SEEN"
_TEL=$(${CLAUDE_PLUGIN_ROOT}/bin/adeel-config get telemetry 2>/dev/null || true)
_TEL_PROMPTED=$([ -f $HOME/.adeel/.telemetry-prompted ] && echo "yes" || echo "no")
_TEL_START=$(date +%s)
_SESSION_ID="$$-$(date +%s)"
echo "TELEMETRY: ${_TEL:-off}"
echo "TEL_PROMPTED: $_TEL_PROMPTED"
mkdir -p $HOME/.adeel/analytics
echo '{"skill":"adeel","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> $HOME/.adeel/analytics/skill-usage.jsonl 2>/dev/null || true
# zsh-compatible: use find instead of glob to avoid NOMATCH error
```

If `PROACTIVE` is `"false"`, do not proactively suggest adeel skills — only invoke
them when the user explicitly asks. The user opted out of proactive suggestions.

If `LAKE_INTRO` is `no`: Before continuing, introduce the Completeness Principle.
Tell the user: "adeel follows the **Go all the way** principle — always do the complete
Then offer to open the essay in their default browser:

```bash
```

Only run `open` if the user says yes. Always run `touch` to mark as seen. This only happens once.

If `TEL_PROMPTED` is `no` AND `LAKE_INTRO` is `yes`: After the lake intro is handled,
ask the user about telemetry. Use AskUserQuestion:

> Help adeel get better! Community mode shares usage data (which skills you use, how long
> they take, crash info) with a stable device ID so we can track trends and fix bugs faster.
> No code, file paths, or repo names are ever sent.
> Change anytime with `adeel-config set telemetry off`.

Options:
- A) Help adeel get better! (recommended)
- B) No thanks

If A: run `${CLAUDE_PLUGIN_ROOT}/bin/adeel-config set telemetry community`

If B: ask a follow-up AskUserQuestion:

> How about anonymous mode? We just learn that *someone* used adeel — no unique ID,
> no way to connect sessions. Just a counter that helps us know if anyone's out there.

Options:
- A) Sure, anonymous is fine
- B) No thanks, fully off

If B→A: run `${CLAUDE_PLUGIN_ROOT}/bin/adeel-config set telemetry anonymous`
If B→B: run `${CLAUDE_PLUGIN_ROOT}/bin/adeel-config set telemetry off`

Always run:
```bash
touch $HOME/.adeel/.telemetry-prompted
```

This only happens once. If `TEL_PROMPTED` is `yes`, skip this entirely.

## Contributor Mode

If `_CONTRIB` is `true`: you are in **contributor mode**. At the end of each major workflow step, rate your adeel experience 0-10. If not a 10 and there's an actionable bug or improvement — file a field report.

**File only:** adeel tooling bugs where the input was reasonable but adeel failed. **Skip:** user app bugs, network errors, auth failures on user's site.

**To file:** write `$HOME/.adeel/contributor-logs/{slug}.md`:
```
# {Title}
**What I tried:** {action} | **What happened:** {result} | **Rating:** {0-10}
## Repro
1. {step}
## What would make this a 10
{one sentence}
**Date:** {YYYY-MM-DD} | **Version:** {version} | **Skill:** /{skill}
```
Slug: lowercase hyphens, max 60 chars. Skip if exists. Max 3/session. File inline, don't stop.

## Completion Status Protocol

When completing a skill workflow, report status using one of:
- **DONE** — All steps completed successfully. Evidence provided for each claim.
- **DONE_WITH_CONCERNS** — Completed, but with issues the user should know about. List each concern.
- **BLOCKED** — Cannot proceed. State what is blocking and what was tried.
- **NEEDS_CONTEXT** — Missing information required to continue. State exactly what you need.

### Escalation

It is always OK to stop and say "this is too hard for me" or "I'm not confident in this result."

Bad work is worse than no work. You will not be penalized for escalating.
- If you have attempted a task 3 times without success, STOP and escalate.
- If you are uncertain about a security-sensitive change, STOP and escalate.
- If the scope of work exceeds what you can verify, STOP and escalate.

Escalation format:
```
STATUS: BLOCKED | NEEDS_CONTEXT
REASON: [1-2 sentences]
ATTEMPTED: [what you tried]
RECOMMENDATION: [what the user should do next]
```

## Telemetry (run last)

After the skill workflow completes (success, error, or abort), log the telemetry event.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agwacom/adeel](https://github.com/agwacom/adeel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
