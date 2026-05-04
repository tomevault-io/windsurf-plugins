---
trigger: always_on
description: > Read this FIRST. It will save you 10+ minutes per task.
---

# AGENTS.md -- Agent Speed Reference for ROT Codebase

> Read this FIRST. It will save you 10+ minutes per task.

## LOOKUP FILES (grep these, don't scan CLAUDE.md)

| Need | File | Example |
|------|------|---------|
| SQL patterns | `.claude/sql.h` | `grep "WIN_CASE" .claude/sql.h` |
| Routes | `.claude/route.tbl` | `grep "backtest" .claude/route.tbl` |
| Tier gates | `.claude/tier.map` | `grep "gate_backtest" .claude/tier.map` |
| Codebase index | `.claude/rot.idx` | `grep "^signals\|" .claude/rot.idx` |
| Test templates | `.claude/test.gen` | Copy macro, substitute params |
| Update rules | `.claude/diff.rules` | Check before committing |
| Step-by-step | `.claude/hotpath.md` | 5 common tasks as decision trees |
| Import map | `docs/agent-map.compact` | Module exports in compact format |
| Code recipes | `COOKBOOK.md` | 10 copy-paste recipes |
| Code templates | `docs/agent-patterns.toml` | TOML code gen templates |

---

## 1. SPEED INDEX

```
Test all:       python -m pytest tests/ -x --tb=short
Test one file:  python -m pytest tests/test_foo.py -v
Test one func:  python -m pytest tests/test_foo.py::test_bar -v
Server:         python -m rot.app.server
Lint:           ruff check src/ tests/
Line length:    100 chars (ruff, pyproject.toml)
Python:         >=3.10 (deployed 3.12)
Async mode:     auto (pyproject.toml: asyncio_mode = "auto") -- NO @pytest.mark.asyncio needed
DB fixture:     async def db(tmp_path) -> Database: connect/yield/close
Templates:      extend base.html, use {% block content %}...{% endblock %}
Static JS:      /static/js/ for Chart.js, HTMX, HTMX-WS (self-hosted, no CDN)
```

---

## 2. FILE OWNERSHIP -- Who Owns What

### Pipeline (sync thread)
```
src/rot/app/runner.py          -- PipelineRunner orchestration
src/rot/ingest/                -- all data ingestion
src/rot/trend/                 -- trend detection + ranking
src/rot/nlp/                   -- NLP analysis (10 modules)
src/rot/extract/               -- event building (NLP + legacy)
src/rot/credibility/           -- ML + heuristic scoring
src/rot/feedback/              -- suppressor (stage 6.5)
src/rot/reasoner/              -- LLM reasoning
src/rot/market/                -- trade building, enrichment, price checks
```

### Web (async FastAPI)
```
src/rot/web/app.py             -- FastAPI factory, route registration
src/rot/web/auth.py            -- JWT/API key/session auth
src/rot/web/tier_gate.py       -- 30 gate functions
src/rot/web/query_cache.py     -- dashboard query cache
src/rot/web/rate_limit.py      -- per-tier rate limiting
src/rot/web/routes/            -- 41 route files
src/rot/web/templates/         -- 39+ Jinja2 templates
src/rot/web/static/            -- self-hosted JS (Chart.js, HTMX)
```

### Storage
```
src/rot/storage/database.py    -- ALL DB operations (27+ tables, 100+ methods)
```

### Analytics engines (pure logic, no DB)
```
src/rot/backtest/              -- 12 modules, backtesting engine
src/rot/unusual/               -- 4 modules, unusual activity detection
src/rot/analysis/              -- 5 modules, sector + correlation
src/rot/macro/                 -- 7 modules, economic calendar + earnings + insider + FOMC
src/rot/agents/                -- 3 modules, autonomous trading agents
src/rot/export/                -- 4 modules, enterprise export + lineage
```

### Alerts
```
src/rot/alerts/                -- 5 modules (dispatcher, discord, email, twitter, webhook)
```

### Server + background loops
```
src/rot/app/server.py          -- uvicorn startup, ALL background loops, signal bridging
```

---

## 3. GOTCHAS -- Things That Bite Agents

1. **Route registration order matters**: Export routes MUST be registered BEFORE signals routes
   in `app.py`. The `/signals/export` path must match before the `/signals/{signal_id}` catch-all.

2. **`_SCHEMA` is safe to re-run**: Uses `CREATE TABLE IF NOT EXISTS`. No need to worry about
   double-creation.

3. **`_MIGRATIONS` is safe to re-run**: Each migration is wrapped in try/except. Adding a column
   that already exists silently succeeds.

4. **All test DB fixtures MUST follow connect/yield/close pattern**:
   ```python
   async def db(tmp_path):
       database = Database(db_path=str(tmp_path / "test.db"))
       await database.connect()
       yield database
       await database.close()
   ```

5. **Templates access via `request.app.state.templates`** -- not a global. Always pass `request`
   as first template context key.

6. **Tier hierarchy**: `free < pro < premium < ultra < enterprise`. Use
   `_PAID_TIERS = ("pro", "premium", "ultra", "enterprise")` from tier_gate.py.

7. **app.state objects** (set in app.py and server.py):
   `db`, `settings`, `signal_queue`, `query_cache`, `templates`,
   `feedback_analyzer`, `agent_engine`, `email_alerter`

8. **Frozen dataclasses**: All types in `rot.core.types` are `@dataclass(frozen=True)`.
   To modify, use `dataclasses.replace(event, confidence=0.9)`.

9. **JSON blob columns**: `market_data`, `reasoning`, `trade_idea`, `event_data` in signals table
   are TEXT columns containing JSON. Use `json.loads()` / `json.dumps()`.

10. **Win/loss logic**: Only `bullish` and `bearish` stances count as trades. Mixed/unknown are

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mattbusel/Reddit-Options-Trader-ROT-](https://github.com/Mattbusel/Reddit-Options-Trader-ROT-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
