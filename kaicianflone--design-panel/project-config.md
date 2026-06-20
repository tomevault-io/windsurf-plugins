---
trigger: always_on
description: |
---


# /design-panel — Multi-Persona Design Review

> Runs 4 UX/UI designer personas in parallel against a live app, then ranks findings
> via cross-persona voting. Outputs a report and a machine-readable fix plan.
> Pairs with /design-review for shipping the ranked fixes.

## ROLE

You are the **Panel Orchestrator**. You do not author findings yourself, do not score findings yourself, do not "improve" persona prompts on the fly. You:

1. Run pre-flight + arg parsing (Phase 0)
2. Detect app-type with a visible `DETECTED:` line (Phase 1)
3. Capture an evidence pack (Phase 2)
4. Select personas (Phase 3)
5. Dispatch all persona reviews in a single parallel Agent call (Phase 4)
6. Dispatch all voting subagents in a single parallel Agent call (Phase 5)
7. Compute ranking, write `report.md` + `fix-plan.md` (Phase 6)
8. Print artifact paths + an optional gstack hand-off tip (Phase 7)

The fix-plan is a data artifact. Anyone (including the user) can feed it to `/design-review` manually if they want to ship the fixes — this skill never invokes other skills automatically.

## Base directory for this skill

The harness exposes the skill's install directory via the "Base directory" line at the top of the loaded skill content. Persona files live at `<base-dir>/personas/<id>.md`. Reference that path explicitly in Phase 4/5 prompts — do not hardcode an absolute path.

---

## TELEMETRY PREAMBLE (run first)

```bash
# gstack-style telemetry preamble — inlined, not inherited.
_TEL=$(~/.claude/skills/gstack/bin/gstack-config get telemetry 2>/dev/null || echo "off")
_TEL_START=$(date +%s)
_SESSION_ID="$$-$(date +%s)"
_OUTCOME="success"  # default; abort/error gates override before epilogue
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "TELEMETRY: ${_TEL:-off}  SESSION: $_SESSION_ID  BRANCH: $_BRANCH"

# Pending marker — epilogue clears it; if the skill crashes the next gstack
# skill to start finalizes it as outcome=unknown.
mkdir -p ~/.gstack/analytics
echo '{"skill":"design-panel","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","session_id":"'"$_SESSION_ID"'"}' \
  > ~/.gstack/analytics/.pending-"$_SESSION_ID" 2>/dev/null || true

# Local analytics start row (gated on gstack telemetry tier)
if [ "$_TEL" != "off" ]; then
  echo '{"skill":"design-panel","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo unknown)'"}' \
    >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
fi

# Timeline event for skill start. Best-effort; failures are silenced.
if [ -x ~/.claude/skills/gstack/bin/gstack-timeline-log ]; then
  _TL_PAYLOAD=$(jq -nc --arg branch "$_BRANCH" --arg sid "$_SESSION_ID" \
    '{skill:"design-panel",event:"started",branch:$branch,session:$sid}' 2>/dev/null || echo '{}')
  ~/.claude/skills/gstack/bin/gstack-timeline-log "$_TL_PAYLOAD" 2>/dev/null &
fi

# Persist telemetry state to disk so the epilogue can recover it even after
# the shell context is lost (each Bash tool call is a fresh shell).
mkdir -p ~/.gstack/analytics
cat > ~/.gstack/analytics/.tel-design-panel-"$_SESSION_ID".sh <<EOF
export _TEL="$_TEL"
export _TEL_START="$_TEL_START"
export _SESSION_ID="$_SESSION_ID"
export _OUTCOME="$_OUTCOME"
EOF
echo "TEL_STATE: ~/.gstack/analytics/.tel-design-panel-$_SESSION_ID.sh"
```

**Note on gstack dependencies:** If `gstack-config` or `gstack-timeline-log` is missing, the bash blocks above silently fall through to `|| true` paths. The skill still runs, just without telemetry. That's intentional — gstack is recommended but not required.

---

## PHASE 0 — Pre-flight + arg parsing

### 0.1 Parse arguments

The user invocation may include:
- `<url>` — optional. If absent, attempt local dev server detection (see 0.2).
- `--personas <list>` — explicit roster (e.g. `a11y,brand,mobile,conversion`), `+id` to add to defaults, `-id` to remove.
- `--deep` — runs all 8 personas instead of the default 4.
- `--report-only` — skip the Phase 7 "next steps" suggestion.
- `--yes` — non-interactive. Auto-confirms the `--deep` cost prompt.

Capture into shell variables: `URL`, `PERSONAS_OVERRIDE`, `DEEP`, `REPORT_ONLY`, `YES`.

### 0.2 Detect project + dev server (only if no `<url>` given)

Read the current working directory for stack indicators. Infer:

1. The project framework (Next.js, Vite, Rails, Django, etc.)
2. The expected dev URL for that framework's defaults
3. The right command to start the dev server

Then probe the inferred URL. If reachable → use it. If not → tell the user the exact command to start it. Do not port-scan a generic list; do not interrogate the user when the project file already tells us what to do.

```bash
_PROJECT_TYPE="unknown"
_EXPECTED_URL=""
_DEV_CMD=""

if [ -f package.json ]; then
  # Package manager from lockfile
  if   [ -f bun.lockb ] || [ -f bun.lock ]; then _PM="bun"
  elif [ -f pnpm-lock.yaml ];                  then _PM="pnpm"
  elif [ -f yarn.lock ];                       then _PM="yarn"
  else                                              _PM="npm"; fi

  # Framework from deps
  if jq -e '(.dependencies // {}).next // (.devDependencies // {}).next' package.json >/dev/null 2>&1; then
    _PROJECT_TYPE="next";  _EXPECTED_URL="http://localhost:3000"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaicianflone/design-panel](https://github.com/kaicianflone/design-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
