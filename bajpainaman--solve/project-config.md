---
trigger: always_on
description: |
---


# /solve — Strategic Problem Solver (agent-teams + gstack)

This skill **must** run as an agent-teams orchestrator. The lead session does intake +
synthesis + rendering. All framework execution and research is delegated to named teammates
so phases run in parallel, peers challenge each other, and Six Hats spawns 6 concurrent
role-plays. Wired to gstack for telemetry, learnings, repo-mode, question tuning, and
continuous checkpoint commits.

---

## Step 0 — Self-check + gstack preamble (run first, fail fast)

```bash
_TEL_START=$(date +%s)
_SESSION_ID="$$-$_TEL_START"
mkdir -p ~/.gstack/sessions ~/.gstack/analytics .context/solve
touch ~/.gstack/sessions/"$PPID"

# ---------- hard requirements ----------
_FAILURES=()

# Skill installed globally?
if [ ! -d "$HOME/.claude/skills/solve" ]; then
  _FAILURES+=("/solve not installed at ~/.claude/skills/solve. Install:")
  _FAILURES+=("    bash <(curl -fsSL https://raw.githubusercontent.com/bajpainaman/solve/main/install)")
fi

# Agent teams flag enabled?
_AT_FLAG="${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-}"
if [ -z "$_AT_FLAG" ] || [ "$_AT_FLAG" = "0" ]; then
  if [ -f "$HOME/.claude/settings.json" ] && \
     python3 -c "import json,sys;d=json.load(open('$HOME/.claude/settings.json'));sys.exit(0 if d.get('env',{}).get('CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS')=='1' else 1)" 2>/dev/null; then
    echo "AGENT_TEAMS: enabled in settings.json (will apply on next claude restart if not yet active)"
  else
    _FAILURES+=("agent teams not enabled. Run installer or add to ~/.claude/settings.json:")
    _FAILURES+=('    {"env": {"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"}}')
  fi
fi

# claude version >= 2.1.32
_CC_VER=$(claude --version 2>/dev/null | grep -Eo '[0-9]+\.[0-9]+\.[0-9]+' | head -1)
if [ -n "$_CC_VER" ]; then
  awk -v v="$_CC_VER" 'BEGIN{split(v,a,".");exit !(a[1]>2 || (a[1]==2 && a[2]>1) || (a[1]==2 && a[2]==1 && a[3]>=32))}' \
    || _FAILURES+=("claude $_CC_VER too old, need >= 2.1.32 for agent teams")
fi

if [ ${#_FAILURES[@]} -gt 0 ]; then
  printf '\n/solve cannot start. Issues:\n'
  printf '  - %s\n' "${_FAILURES[@]}"
  exit 1
fi
echo "SELFCHECK: ok"

# ---------- gstack preamble ----------
_UPD=$(~/.claude/skills/gstack/bin/gstack-update-check 2>/dev/null || true)
[ -n "$_UPD" ] && echo "$_UPD" || true

# Spawned-session detection (suppresses interactive prompts)
[ -n "$OPENCLAW_SESSION" ] && SPAWNED_SESSION=true || SPAWNED_SESSION=false
echo "SPAWNED_SESSION: $SPAWNED_SESSION"

# User preferences (gstack-config)
_PROACTIVE=$(~/.claude/skills/gstack/bin/gstack-config get proactive 2>/dev/null || echo "true")
_EXPLAIN_LEVEL=$(~/.claude/skills/gstack/bin/gstack-config get explain_level 2>/dev/null || echo "default")
[ "$_EXPLAIN_LEVEL" != "default" ] && [ "$_EXPLAIN_LEVEL" != "terse" ] && _EXPLAIN_LEVEL="default"
_QUESTION_TUNING=$(~/.claude/skills/gstack/bin/gstack-config get question_tuning 2>/dev/null || echo "false")
_TEL=$(~/.claude/skills/gstack/bin/gstack-config get telemetry 2>/dev/null || echo "off")
_CROSS_PROJ=$(~/.claude/skills/gstack/bin/gstack-config get cross_project_learnings 2>/dev/null || echo "false")
_CHECKPOINT_MODE=$(~/.claude/skills/gstack/bin/gstack-config get checkpoint_mode 2>/dev/null || echo "explicit")
_CHECKPOINT_PUSH=$(~/.claude/skills/gstack/bin/gstack-config get checkpoint_push 2>/dev/null || echo "false")

echo "PROACTIVE: $_PROACTIVE"
echo "EXPLAIN_LEVEL: $_EXPLAIN_LEVEL"
echo "QUESTION_TUNING: $_QUESTION_TUNING"
echo "TELEMETRY: $_TEL"
echo "CROSS_PROJECT_LEARNINGS: $_CROSS_PROJ"
echo "CHECKPOINT_MODE: $_CHECKPOINT_MODE"

# Repo mode (solo / collaborative)
source <(~/.claude/skills/gstack/bin/gstack-repo-mode 2>/dev/null) || true
REPO_MODE=${REPO_MODE:-unknown}
echo "REPO_MODE: $REPO_MODE"

# Branch + analytics ping
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"

if [ "$_TEL" != "off" ]; then
  echo '{"skill":"solve","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}' \
    >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
fi
```

If `_UPD` shows `UPGRADE_AVAILABLE <old> <new>`: tell the user "/solve has an update available: <old> -> <new>. Run `solve --update` to refresh." Continue the run; don't block.

If `_UPD` shows `JUST_UPGRADED <from> <to>`: print "Running /solve v{to} (just updated)."

If `SPAWNED_SESSION=true`, the skill is running inside another orchestrator (OpenClaw, etc.):
- Do NOT use AskUserQuestion. Auto-choose recommended options.
- Skip Lake Intro, telemetry consent, routing-rules prompt, and Context Recovery prompts.
- Focus on completing the task and reporting via prose output.

---

## Step 0.5 — Regime Classification (MANDATORY GATE)

**This step gates everything else.** /solve is heavyweight: 5 long-lived teammates + 6 just-in-time hat teammates + 25 frameworks + parallel research. That investment is correct for **strategic decisions** and badly miscalibrated for everything else. This step routes the question to the right shape.

### The classifier

AskUserQuestion (skip if `SPAWNED_SESSION=true`; auto-choose `decision` in that case):

> Before /solve runs the full workflow, what shape is this question?
>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bajpainaman/solve](https://github.com/bajpainaman/solve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
