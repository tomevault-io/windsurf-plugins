---
trigger: always_on
description: **AgentTelemetry** — a local, **stdlib-only** dashboard that reads the logs your AI coding tools already write
---

# AGENTS.md

**AgentTelemetry** — a local, **stdlib-only** dashboard that reads the logs your AI coding tools already write
to this machine and shows tokens, estimated cost, and breakdowns by model / day / tool /
project / hour. Nothing leaves the machine unless the user turns on sharing (see "Your
devices"). Nothing to install.

```bash
python3 dashboard.py            # http://127.0.0.1:7878
```
Flags: `--port`, `--host`, `--interval`, `--rebuild`. First run parses everything (~30–60s
with big Codex logs), then caches; later refreshes are incremental. If asked to "run the
dashboard", check `curl -s -o /dev/null -w '%{http_code}' http://127.0.0.1:7878/api/data`
first — it may already be up.

## Layout

| File | Role |
|---|---|
| `dashboard.py` | stdlib `http.server`. Serves `/`, `/static/*`, `/chart.js`, `/manifest.json`, `/sw.js`, `/api/{data,storage,refresh,settings,cache,update,devices}`. Owns the cache, the aggregate merge, `_cost()` and the device sharing listener. |
| `parser.py` | `discover()` lists log files; `update_file()` routes each to a `parse_*`. Holds `PRICING` and the model-name normalizers. |
| `static/core.js` | `SRC`/`ORDER`, state `S`, formatting, date ranges, filtering. |
| `static/charts.js` | Chart.js theming, `mk()`/`hbar()`/`areaDS()`, calendar + heatmap SVG. |
| `static/views.js` | The eight views, controls, events, boot. |
| `index.html` | Shell only: sidebar (device name, tabs, live/refresh/theme/settings), page header (period, range, metric, filters), card markup, an SVG icon sprite. |

Eight tabs in the sidebar: Overview · Cost · Models · Tools · Projects · Sessions ·
**Optimize** · Storage. Tab lives in `location.hash`. **Every filter lives in the query
string too**, so a reload, bookmark or pasted link opens the same view: `filtersFromURL()`
reads it at boot and `filtersToURL()` writes it after every `renderAll()`, both in `core.js`.
The params are `?range=` (a preset, or `custom` with `&from=&to=` as YYYY-MM-DD),
`?metric=` (`tokens|cost|messages|time`), `?rate=out`, `?tools=`, `?providers=`, `?models=`,
`?projects=`, `?ides=`, `?devices=` (each repeated once per value), `?exact=1` and `?q=`.
Only what differs from the default is written, params that aren't the filters' own are left
alone, and a change replaces the history entry the way switching tab does. `?theme=` and
`?side=collapsed|open` preset the look (handy for headless screenshots). **A new filter goes
in `S` and in both functions**, or it silently resets on reload; sort order, table twins,
muted series and expanded rows are display state and stay out of the URL. The sidebar
collapses to an icon rail (`[`, remembered as `aiu.side`); below 900px it becomes a top bar
instead.

**The controls are a sentence**, not a toolbar: "**Tokens** from **all tools** over **the last
30 days** ‹ ›". Each bold phrase is a `.dd` that opens its menu (`#metricPanel`,
`#filtersPanel`, `#rangePanel`); `periodPhrase()` supplies the connecting word ("over",
"in", "across", or none for "today" / "this month"); ‹ › (`stepPeriod`, keys `,` `.`) move
the window back or forward by its own length as a custom range, stopping at today. Tokens
are the default measure — the Overview hero follows it, and cost is one of its tiles. Every chart has a table twin (`.tv[data-tv]`, `chartTable()`), and each view
leads with one hero figure — keep it that way rather than adding a second.

The device name at the top of the sidebar is `DEVICE` in `dashboard.py`: the OS's own name
for the machine (`scutil --get ComputerName` on macOS, `%COMPUTERNAME%`, `/etc/machine-info`
`PRETTY_HOSTNAME`), falling back to the bare hostname. It's in the payload as `device`.

Aggregates are keyed `records["date\tmodel"]`, `tools["date\tname"]`,
`hourly["date\thour"]` — **every dimension carries a date** so the UI can filter by range.
In the payload every usage row (records, sessions, activity, tools, hourly, ctx, skills,
reads) also carries a `device` id; see "Your devices".

## Rules

1. **Stdlib only, offline.** No runtime dependencies. Vendor any JS (Chart.js already is).
2. **Never commit `.usage_cache.json`**, `server.log`, `.peers.json` or `.peers/` — that's
   the user's own prompts, projects, costs and pairing codes. A fresh clone must start empty.
3. **Never hardcode a path.** Derive from `HOME` / `%APPDATA%` / `%LOCALAPPDATA%` /
   `$XDG_*`. Split path components with `_leaf()` (handles `/` and `\`) — logs written on
   one OS get read on another.
4. **Commits are authored by the repo owner alone.** Never add a `Co-authored-by:` trailer.
5. **Costs are estimates** at API list prices; subscription users pay nothing per token.
   Keep that framing. Verify any price against vendor docs — never guess.
6. **Attribute by tool, not by model.** A Claude model run inside Copilot counts as Copilot.
7. **Bump `CACHE_VERSION`** whenever an aggregate's shape changes.
8. **Never `except Exception: pass` around a parser.** A swallowed error is
   indistinguishable from "the user doesn't have this tool" — that's how a `TypeError`
   once made the whole opencode parser silently yield nothing. Write to stderr.
9. **Any write endpoint goes through `Handler._csrf_ok()`.** There's no auth, so any page

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uttamdeb/agent-telemetry](https://github.com/uttamdeb/agent-telemetry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
