---
trigger: always_on
description: wArrden is a scheduled job runner that searches for missing/upgrade items and cleans stuck queue entries on Sonarr/Radarr instances. This file defines console log output formatting standards.
---

# AGENTS

## Purpose

wArrden is a scheduled job runner that searches for missing/upgrade items and cleans stuck queue entries on Sonarr/Radarr instances. This file defines console log output formatting standards.

## Naming

The application is named **wArrden** (camelCase with capital `A`). It must **never** be referred to as "warden", "the warden", or "The warden" in any user-facing text: documentation, changelogs, commit messages, UI output, or log messages. The letter case must always be `wArrden`.

## Log Level & Output

Set via `logLevel` in `config.yaml`:

| `logLevel` | Messages Shown |
|---|---|
| `debug` | DEBUG + INFO + WARN + ERROR (most verbose) |
| `info` (default) | INFO + WARN + ERROR |
| `warning` | WARN + ERROR |
| `error` | ERROR only (least verbose) |

All log messages go through `OutputService` methods and info writers to `Console.Out`. **`Console.Error` (`stderr`) and direct `Console.WriteLine` calls are forbidden for log output.** Containers (Docker, etc.) read stdout/stderr through separate pipes and merge them without preserving write order, which would cause tree-structured output to interleave. Writing everything to a single stream avoids this.

## Log Format Rules

### Shared Rules

- **Stats always come before Results** in all info-level job output.

### Header

Every log entry starts with a header line:

```
[HH:mm:ss LVL] [context]
```

Where:
- `HH:mm:ss` is the 24-hour timestamp of invocation
- `LVL` is the log level marker: `DEBUG`, `INFO`, `WARN`, or `ERROR`
- `context` is `instance.job` for job output, or a system context like `warden.config`, `warden.scheduler`, `cli`

### Tree Structure

All log output uses Unicode box-drawing characters to form a tree:

- ` ├─ ` — intermediate child (not the last)
- ` └─ ` — final child (last in the tree level)
- ` │  ` — vertical connector for nested content under intermediate parent
- `    ` — spacing for nested content under final parent
- ` • ` — bullet for list items

### Debug (DEBUG)

Single message:

```
[07:45:01 DEBUG] [warden.config]
 └─ Loaded config from data/config.yaml
```

With detail:

```
[07:45:01 DEBUG] [series.missing]
 ├─ Fetched 45 wanted episodes
 └─ Cooldown filter: 12 on cooldown, 33 eligible, 3 selected
```

### Warning (WARN)

```
[07:45:01 WARN] [series.missing]
 └─ No enabled indexers — search skipped

[07:45:01 WARN] [series.missing]
 ├─ Search trigger failed for The Boys (2019) - S01E01 - The Name of the Game
 └─ HttpRequestException: Connection refused (192.168.1.100:8989)
```

### Error (ERROR)

```
[07:45:01 ERROR] [series.missing]
 ├─ Missing search job failed
 └─ HttpRequestException: No route to host

[07:45:01 ERROR] [warden.scheduler]
 ├─ Scheduled task error
 └─ InvalidOperationException: Unknown instance type: unknown
```

### Info (INFO) — Search Jobs (missing / upgrade)

```
[07:45:01 INFO] [sonarr.missing]
 ├─ Cleaning cooldown entries
 ├─ Fetching wanted episodes
 ├─ Applying cooldown filters
 └─ Stats:
    • Total Items:   4
    • On Cooldown:   4
    • Eligible:      0
    • Search Limit:  2
    • Result:        No search performed
```

When items are searched, stats come before results:

```
[07:45:01 INFO] [sonarr.missing]
 ├─ Cleaning cooldown entries
 ├─ Fetching wanted episodes
 ├─ Applying cooldown filters
 ├─ Searching 3 items
 ├─ Stats:
 │  • Total Items:   10
 │  • On Cooldown:   2
 │  • Eligible:      8
 │  • Search Limit:  3
 │  • Result:        Searched 3
 └─ Results:
    • The Boys (2019) - S01E01 - The Name of the Game
    • The Boys (2019) - S01E02 - Cherry
    • Game of Thrones (2011) - S01E01 - Winter Is Coming
```

**Rules:**
- Full stats are shown even when no wanted items are found
- `Result` field: `"No wanted items found"` (0 total), `"No search performed"` (0 searched), or `"Searched N"`
- For Radarr, use `"Fetching wanted movies"` instead of `"Fetching wanted episodes"`

### Info (INFO) — Queue Cleanup Job (queue)

No warning-status items:

```
[07:45:01 INFO] [sonarr.queue]
 └─ Stats:
    • Total Queue:   150
    • Result:        No warning queue items detected
```

With `remove` action matches (stats before results):

```
[07:45:01 INFO] [sonarr.queue]
 ├─ Stats:
 │  • Total Queue:   150
 │  • Warnings:      5
 │  • Matched:       2
 │  • Result:        Removed 2
 └─ Results:
    • The Boys (2019) - S01E01 - The Name of the Game  NOT_QUALITY_UPGRADE
    • Game of Thrones (2011) - S02E03 - Valar Morghulis  SAMPLE
```

With `removeAndBlocklist` action matches:

```
[07:45:01 INFO] [sonarr.queue]
 ├─ Stats:
 │  • Total Queue:   80
 │  • Warnings:      8
 │  • Matched:       4
 │  • Result:        Blocklisted 4
 └─ Results:
    • Game of Thrones (2011) - S02E03 - Valar Morghulis  NO_FILES_ELIGIBLE
```

With mixed actions:

```
[07:45:01 INFO] [sonarr.queue]
 ├─ Stats:
 │  • Total Queue:   100
 │  • Warnings:      10
 │  • Matched:       5
 │  • Result:        Removed 3, Blocklisted 2
 └─ Results:
    • Euphoria.US.S01E02.1080p.AMZN.WEB-DL-Obfuscated  UNEXPECTED_ERROR
    • Game of Thrones (2011) - S02E03 - Valar Morghulis  NO_FILES_ELIGIBLE
    • The Boys (2019) - S01E01 - The Name of the Game  NOT_QUALITY_UPGRADE
```

**Rules:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neurekadev/warrden](https://github.com/neurekadev/warrden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
