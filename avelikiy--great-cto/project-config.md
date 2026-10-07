---
trigger: always_on
description: Manage an agent like an employee — review its record, generate its evals, evolve its prompt against held-out evals, or retire it.
---

<!-- great_cto-managed -->

You are the great_cto `/agent` command — one command for an agent's working life.
An agent is managed like an employee: you look at its record, write the exam it is
held to, change how it works only when the exam says the change is better, and let
it go when nobody needs it — keeping the file on it.

| Subcommand | When | What you get |
|---|---|---|
| `review [<name>\|all]` | weekly, after an incident, before a retire | a scorecard: invocations, cost, pass-rate, failure modes, prompt-tuning suggestions |
| `evals <name> [--count N]` | before tuning a prompt, or when an agent has no evals | `tests/eval/EVAL-<name>-*.md` with tuning + holdout cases |
| `evolve <name> [--lesson "…"]` | a lesson says the prompt should change | a candidate prompt, gated on held-out evals — PROMOTED or REJECTED |
| `retire <name>` | idle 90 days, superseded, or misbehaving | the prompt archived to `agents/_retired/`, verdicts kept, reversible |

---

## Dispatch by argument

```bash
SUB="${1:-review}"
case "$SUB" in
  review)  SUBCOMMAND=review; [ $# -gt 0 ] && shift ;;  # one agent, or `all` / no name = the whole workforce
  evals)   SUBCOMMAND=evals;  shift ;;                  # EVAL-*.md cases (tuning + holdout) from the agent's prompt
  evolve)  SUBCOMMAND=evolve; shift ;;                  # lesson → candidate prompt → holdout gate → PROMOTE | REJECT
  retire)  SUBCOMMAND=retire; shift ;;                  # archive the prompt, keep the verdicts
  *)       SUBCOMMAND=review ;;                         # `/agent <name>` or `/agent --idle` reads as a review
esac
# After the shift, "$@" holds only the subcommand's own arguments.
```

---

## Subcommand: review — performance scorecard

`/agent review` is the performance scorecard for the AI workforce. Two modes:

- **List mode** (no name, or `all`): summary table of all agents — invocations, cost, pass-rate, last activity
- **Detail mode** (`/agent review <name>`): drill-down scorecard with cost analysis, failure modes, prompt-tuning suggestions

Inspired by human 1:1s, but adapted for LLM agents: data-driven, periodic, focused on observable outcomes (verdicts) rather than emotional check-in.

### When to use

- **Weekly:** `/agent review` to see who's pulling weight
- **After incident:** `/agent review <agent>` if the agent missed something critical
- **Before retiring:** `/agent review <name> --since 90d` to confirm low usage
- **For cost optimization:** `/agent review --top-cost` to find expense outliers

### Step 1 — Parse args

```bash
source .great_cto/env.sh 2>/dev/null || export PATH="/opt/homebrew/bin:$HOME/.local/bin:/usr/local/bin:$PATH"

# Default window: last 30 days
SINCE_DAYS=30
AGENT_NAME=""
TOP_COST=0
IDLE_ONLY=0

# Parse arguments — first non-flag is agent name
for arg in "$@"; do
  case "$arg" in
    --since)        ;; # next arg is value
    --since=*)      SINCE_DAYS=$(echo "$arg" | sed 's/--since=//; s/d$//') ;;
    --top-cost)     TOP_COST=1 ;;
    --idle)         IDLE_ONLY=1 ;;
    --*)            ;; # unknown flag, ignore
    all)            ;; # `all` = list mode, same as no name
    *)              [ -z "$AGENT_NAME" ] && AGENT_NAME="$arg" ;;
  esac
done

# Compute since-timestamp (cross-platform: macOS BSD date + GNU date)
SINCE_TS=$(date -u -v -${SINCE_DAYS}d +%Y-%m-%dT%H:%M:%SZ 2>/dev/null || \
           date -u -d "${SINCE_DAYS} days ago" +%Y-%m-%dT%H:%M:%SZ 2>/dev/null)

VERDICTS_DIR=~/.great_cto/verdicts
COST_LOG=~/.great_cto/cost-history.log
[ -d "$VERDICTS_DIR" ] || VERDICTS_DIR=.great_cto/verdicts
[ -f "$COST_LOG" ]     || COST_LOG=.great_cto/cost-history.log

if [ ! -d "$VERDICTS_DIR" ]; then
  echo "No verdicts found yet. /agent review activates after agents emit verdicts."
  echo "Path checked: $VERDICTS_DIR"
  exit 0
fi
```

### Step 2 — List mode (no agent name)

If `$AGENT_NAME` is empty, output a table of all agents:

```bash
if [ -z "$AGENT_NAME" ]; then
  echo "## Agent workforce — last $SINCE_DAYS days"
  echo ""
  echo "| Agent | Invocations | APPROVED | Cost | Avg/inv | Last activity |"
  echo "|-------|------------:|---------:|-----:|--------:|---------------|"

  for log in "$VERDICTS_DIR"/*.log; do
    [ -f "$log" ] || continue
    AGENT=$(basename "$log" .log)

    # Filter to since-window
    RECENT=$(awk -v ts="$SINCE_TS" '$1 > ts' "$log")
    INVOC=$(echo "$RECENT" | grep -c .)
    [ "$INVOC" = "0" ] && [ "$IDLE_ONLY" = "0" ] && continue   # skip empty unless --idle

    APPROVED=$(echo "$RECENT" | grep -c APPROVED)
    PASS_RATE=$([ "$INVOC" -gt 0 ] && echo "scale=0; $APPROVED * 100 / $INVOC" | bc || echo 0)

    # Per-agent cost (filter cost-history.log for this agent name)
    COST=$(grep -E "^[^ ]+ agent=$AGENT " "$COST_LOG" 2>/dev/null | \
      awk -v ts="$SINCE_TS" '$1 > ts {
        for (i=1;i<=NF;i++) if ($i ~ /^cost[-_]?usd[=:]/) { gsub(/cost[-_]?usd[=:]/, "", $i); sum += $i }
      } END { printf "%.2f", sum+0 }')

    AVG=$(echo "scale=2; $COST / $INVOC" | bc 2>/dev/null || echo "0.00")
    LAST=$(echo "$RECENT" | tail -1 | awk '{print $1}')

    printf "| %s | %d | %d%% | \$%s | \$%s | %s |\n" "$AGENT" "$INVOC" "$PASS_RATE" "$COST" "$AVG" "$LAST"
  done

  if [ "$IDLE_ONLY" = "1" ]; then
    echo ""

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [avelikiy/great_cto](https://github.com/avelikiy/great_cto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
