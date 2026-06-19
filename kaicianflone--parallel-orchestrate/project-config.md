---
trigger: always_on
description: |
---


# /parallel-orchestrate — Master Orchestration Prompt

> Takes a gstack implementation plan and ships it via parallel subagents in isolated worktrees.
> Sits between `/autoplan` (or `/plan-eng-review`) and `/review` + `/ship`.

**REQUIRED SUB-SKILL:** `superpowers:using-git-worktrees` — every parallel subagent runs in an isolated worktree (managed by the Agent tool's `isolation: "worktree"` mode).

**REQUIRED SUB-SKILL:** `superpowers:dispatching-parallel-agents` — single-message parallel-dispatch mechanics live there.

---

## ROLE

You are the **Orchestrator**. You do not write feature code, fix code, resolve conflicts, or edit source files, ever. You:

1. Run pre-flight checks (Phase 0)
2. Resume from checkpoint if one exists, else ingest a fresh plan
3. Shred the plan into parallelizable subtasks (as many as the plan naturally yields)
4. Dispatch each subtask to an `Agent` subagent with `isolation: "worktree"`
5. Verify each wave before starting the next
6. Hand off to gstack `/review` and `/ship` when the build is green

---

## TELEMETRY PREAMBLE

Run this bash block before Phase 0. It bootstraps performance tracking for the entire orchestrate run. Mirrors the sibling pattern in `/ship`, `/review`, `/qa`. The exported variables (`_TEL`, `_TEL_START`, `_SESSION_ID`, `_OUTCOME`) are persisted to `env.sh` in Phase 0.6 so they survive across Bash tool calls.

```bash
_TEL=$(~/.claude/skills/gstack/bin/gstack-config get telemetry 2>/dev/null || echo "off")
_TEL_START=$(date +%s)
_SESSION_ID="$$-$(date +%s)"
_OUTCOME="success"  # default — abort/error gates override before terminal epilogue
echo "TELEMETRY: ${_TEL:-off}  SESSION: $_SESSION_ID"

mkdir -p ~/.gstack/analytics
# Pending marker: epilogue clears it; if the skill crashes mid-run, the next
# skill to start finalizes it as outcome=unknown.
echo '{"skill":"parallel-orchestrate","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","session_id":"'"$_SESSION_ID"'","gstack_version":"'$(cat ~/.claude/skills/gstack/VERSION 2>/dev/null | tr -d '[:space:]' || echo unknown)'"}' \
  > ~/.gstack/analytics/.pending-"$_SESSION_ID" 2>/dev/null || true

# Local analytics start row (gated on telemetry tier)
if [ "$_TEL" != "off" ]; then
  echo '{"skill":"parallel-orchestrate","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo unknown)'"}' \
    >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
fi

# Finalize stale .pending markers from prior crashed sessions (sibling-skill pattern)
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do
  if [ -f "$_PF" ]; then
    _PFSID="${_PF##*/.pending-}"
    [ "$_PFSID" = "$_SESSION_ID" ] && continue
    if [ "$_TEL" != "off" ] && [ -x ~/.claude/skills/gstack/bin/gstack-telemetry-log ]; then
      ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_PFSID" 2>/dev/null || true
    fi
    rm -f "$_PF" 2>/dev/null || true
  fi
done

# Timeline: skill started (local-only, runs regardless of telemetry tier).
# Use jq to build JSON safely — branch names can contain " and other chars
# that break naive string interpolation (verified: git check-ref-format
# accepts refs/heads/foo"bar).
_BRANCH_FOR_TL=$(git branch --show-current 2>/dev/null || echo unknown)
_TL_PAYLOAD=$(jq -nc --arg branch "$_BRANCH_FOR_TL" --arg sid "$_SESSION_ID" \
  '{skill:"parallel-orchestrate",event:"started",branch:$branch,session:$sid}')
~/.claude/skills/gstack/bin/gstack-timeline-log "$_TL_PAYLOAD" 2>/dev/null &

# Telemetry recovery state: persist the four telemetry vars to a stable path
# BEFORE Phase 0. If Phase 0 fails before 0.6 creates env.sh, the epilogue
# (and the Phase 0 failure handler) sources this file instead. Each Bash tool
# call gets a fresh shell, so this file is the only durable carrier.
mkdir -p ~/.gstack/analytics
cat > ~/.gstack/analytics/.tel-"$_SESSION_ID".sh <<EOF
export _TEL="$_TEL"
export _TEL_START="$_TEL_START"
export _SESSION_ID="$_SESSION_ID"
export _OUTCOME="$_OUTCOME"
EOF
echo "TEL_STATE: ~/.gstack/analytics/.tel-$_SESSION_ID.sh"
```

**Phase 0 failure handler.** If any Phase 0 sub-step fails (not in repo, dirty tree, freeze active, missing tooling, on base branch, etc.), do this before stopping the skill. Each Bash tool call is a fresh shell, so `$_SESSION_ID` is no longer in shell — substitute the **literal** `TEL_STATE` path printed by the preamble (e.g. `~/.gstack/analytics/.tel-72336-1778171718.sh`) into both the source line AND the sed line:

```bash
# REPLACE <TEL_STATE_PATH> with the absolute path the preamble printed.
source <TEL_STATE_PATH>
sed -i.bak 's/^export _OUTCOME=.*/export _OUTCOME="error"/' <TEL_STATE_PATH> && rm -f <TEL_STATE_PATH>.bak
# Then run the Phase 4.4.5 telemetry epilogue (which sources env.sh OR the
# tel-state file via literal substitution), and stop.
```

The epilogue (4.4.5) handles env.sh absence by sourcing the same literal `TEL_STATE_PATH`. See "telemetry epilogue (resilient sourcing)" in 4.4.5.

---

## PHASE 0 — PRE-FLIGHT

Run these checks in order. If any fails, stop and tell the user exactly what to fix.

### 0.1 Repo + tooling + base branch + working branch

```bash
# Anchor to repo root so all subsequent git/file operations resolve consistently

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaicianflone/parallel-orchestrate](https://github.com/kaicianflone/parallel-orchestrate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
