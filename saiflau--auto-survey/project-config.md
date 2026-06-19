---
trigger: always_on
description: Run an autonomous, long-lived literature-survey workflow that maintains state across turns. Use when user says \"自动调研\", \"auto survey\", \"长跑调研\", \"start a survey on X\", \"resume my survey\", or wants a recoverable multi-phase research-survey loop on top of arxiv / research-lit. Also covers a stateless enhancement sub-mode for existing paper notes — use when user says \"迁移笔记到 Notion\", \"把 vault 的论文搬过去\", \"补充方法解释\", \"AWQ 解释得不清楚\", \"加公式说明\", \"fix Notion paper cross-links\", \"修笔记里的链接\", or
---


# auto-survey: long-running literature-survey orchestrator

Drive a multi-phase survey via a state machine in `.auto-survey/state.json`. Each invocation advances **at most one phase**, persists state, and either schedules itself to wake up (in `/loop` dynamic mode) or asks the user to `/auto-survey resume`.

Designed to run identically under Claude Code and Codex CLI. The body below uses only generic tools (shell, file I/O, web search, optional MCP). The only host-specific piece is the wake-up step at the end, which degrades gracefully when the host doesn't support it.

## Context: $ARGUMENTS

## Safety Rules — READ FIRST

- **NEVER** `sudo`, `rm -rf`, `rm -r`, recursive deletion, or destructive git ops.
- **NEVER** modify files outside the current working directory's `.auto-survey/` folder, unless writing to an explicitly user-configured Obsidian vault or Notion parent page.
- **NEVER** download more than `arxiv_max_download` PDFs per phase.
- If a step *would* require any of the above, STOP and report.

## Engine Layout

The Python engine lives next to this file:

```
SKILL_DIR/
  scripts/
    init.py        # initialise .auto-survey/ workspace
    state.py       # atomic state.json read/write + CLI subcommands
    budget.py      # check abort conditions
    score_paper.py # heuristic relevance scoring
    obsidian_io.py # render templates → markdown files
  templates/       # state.template.json, paper_note.template.md, ...
```

To resolve `SKILL_DIR` from anywhere:

```bash
SKILL_DIR=$(python3 -c "
import pathlib
for p in [
    pathlib.Path.home() / '.claude' / 'skills' / 'auto-survey',
    pathlib.Path.home() / '.codex'  / 'skills' / 'auto-survey',
]:
    if (p / 'scripts' / 'state.py').exists():
        print(p); break
")
```

Use `$SKILL_DIR/scripts/<name>.py` in subsequent commands.

## Step 0 — Parse $ARGUMENTS

Detect the sub-command:

| First token | Meaning |
|---|---|
| `resume` (or empty) | Advance one phase based on existing state.json |
| `status` | Print state summary; no writes |
| `abort` | Set status=aborted; refuse future advances |
| anything else | Treat as a topic; init a new workspace |

Also extract `— flag: value` overrides (same style as `research-lit`):

| Flag | Effect |
|---|---|
| `— max_papers: N` | sets `config.max_papers_to_read` |
| `— max_iterations: N` | sets `budget.max_iterations` |
| `— deadline: ISO8601` | sets `budget.deadline` |
| `— sources: a,b,c` | restrict to subset of `arxiv,web,local,zotero,obsidian` |
| `— no_download` | sets `config.arxiv_download = false` |
| `— notion_parent: <URL_or_ID>` | enables Notion sync under that parent page |

## Step 1 — Load or initialise state

```bash
if [ -f .auto-survey/state.json ]; then
  python3 "$SKILL_DIR/scripts/state.py" show --summary
else
  # init mode (only allowed when first token is a topic, not resume/status/abort)
  python3 "$SKILL_DIR/scripts/init.py" "TOPIC" \
    --max-papers $MAX_PAPERS --max-iterations $MAX_ITERS \
    --obsidian-configured $OBSIDIAN_AVAILABLE \
    ${NOTION_PARENT:+--notion-parent "$NOTION_PARENT"}
fi
```

**Stale-state guard** (mirrors `auto-review-loop`):

- If `status == "completed"` or `"aborted"`: refuse `resume`; tell user to `rm -rf .auto-survey/` first.
- If `status == "in_progress"` AND `updated_at` older than 24h: warn but continue (the user may have abandoned it).

**Obsidian detection**: try one Obsidian MCP call (`mcp__obsidian-vault__*` or `mcp__obsidian__*`). If it succeeds, set `state.obsidian.configured = true` and remember the vault folder `Research/<topic_slug>/`. If it fails, fall back to local `.auto-survey/notes/`.

**Notion detection**: only enabled if the user passed `— notion_parent: <URL_or_ID>` (already stored at init time in `state.notion.parent_page_id`). If set AND a Notion MCP tool is available (`mcp__plugin_Notion_notion__*`), keep `state.notion.configured = true`. If the user later wants to add Notion to an existing survey, edit `state.notion.parent_page_id` manually and re-`resume`. The Papers database (`state.notion.papers_database_id`) and Survey page (`state.notion.survey_page_id`) are created lazily on first write, not here.

## Step 2 — Check budget

```bash
python3 "$SKILL_DIR/scripts/budget.py"
# Exit 0  → continue
# Exit 10/11/12/13/14 → abort: read the JSON for reason
```

If abort: skip to Step 4 (done) and write final report from whatever has been collected.

## Step 3 — Dispatch by phase

The state machine. Read `state.phase` and execute the matching block. **Each block does ONE small step then returns to Step 5** (don't try to loop multiple phases in one turn — that's what wake-up is for).

### Phase: `init`

Just transition to `keyword_expansion`.
```bash
python3 "$SKILL_DIR/scripts/state.py" phase keyword_expansion --reason "init complete"
```

### Phase: `keyword_expansion`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SaifLau/auto-survey](https://github.com/SaifLau/auto-survey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
