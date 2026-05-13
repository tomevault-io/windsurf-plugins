---
trigger: always_on
description: > This document serves as the single source of truth for all agents working on the BiliTicketMonitor project. It documents both the C++ core (`src/main.cpp`) and the Python stock monitor (`extra/HSR_Land_Stock_Monitor.py`), with a detailed comparison and merge strategy.
---

# BiliTicketMonitor Architecture Knowledge Base

> This document serves as the single source of truth for all agents working on the BiliTicketMonitor project. It documents both the C++ core (`src/main.cpp`) and the Python stock monitor (`extra/HSR_Land_Stock_Monitor.py`), with a detailed comparison and merge strategy.

---

## 1. Project Overview

**BiliTicketMonitor** is a Bilibili ticketing event monitor that polls ticket availability and notifies when stock status changes. It consists of two independent implementations in the same repository:

| Component | Language | Lines | File |
|-----------|----------|-------|------|
| Core Monitor | C++20 | 852 | `src/main.cpp` |
| Stock Monitor | Python 3 | 313 | `extra/HSR_Land_Stock_Monitor.py` |

### Build System
- CMake-based build via `CMakeLists.txt`
- Dependencies: `libcurl` (HTTP), `cJSON` (JSON parsing, header-only, bundled in `src/`)
- Output: `build/BiliTicketMonitor` (Linux)

### Runtime Dependencies
- C++ side: libcurl, libcJSON (embedded header), C++20 standard library
- Python side: `requests`, `urllib3`, `colorama` (installed via `pip install -r requirements.txt`)

### Author
- FriendshipEnder (https://space.bilibili.com/180327370)
- Project URL: https://github.com/fsender/BiliTicketMonitor

---

## 2. main.cpp Deep Analysis

### Architecture

The C++ monitor (`src/main.cpp`, v2.0.0) is a single-file, single-threaded application structured around three core types:

#### Config Class (lines 88-321)
- Static class managing all configuration via `static` members
- `config.txt` format: 7 lines (ticket_id, ticket_no, bat_path, refresh_interval_ms, timeout_ms, api_base_url, user_agent_header) + comment section
- Supports CLI arguments: `--id`, `--ticket-no`, `--interval`, `--script` (paired-key format: `--id 102194 --ticket-no 1 --interval 300 --script bhyg.bat`)
- `checkconf()` reads and validates `config.txt`. On failure, deletes old config and prompts interactive input
- Default ticket ID: `102194` (BW2025)

#### HttpResponse Struct (lines 323-328)
```cpp
struct HttpResponse {
    string data;      // Response body (raw JSON)
    long status_code; // HTTP status code
    string error;     // libcurl error string on failure
};
```

#### Monitor Class (lines 547-619)
- Single-threaded polling loop: `run_monitor()` continuously calls `http_get()` then `process_data()`
- Uses `last_data` (vector<vector<string>>) for diff detection to avoid redundant table renders
- `selling` flag set when the monitored ticket's status becomes "预售中" (on sale)
- On `selling == true`, executes the batch script via `system(Config::BATPATH.c_str())`
- Error handling: 412 triggers critical stop; other errors set `healthy = false` temporarily
- `show_table()` renders a formatted TUI with ANSI color codes for status display

#### HTTP Layer (lines 357-387)
- `http_get()` wraps libcurl for synchronous GET requests
- `WriteCallback` is a C-style curl write function that appends to a `std::string`
- Uses `curl_slist` for custom headers; SSL verification disabled
- Timeout configurable via `Config::TIMEOUT` (default 10000ms)

#### JSON Parsing (lines 390-461)
- `process_data()` uses cJSON to parse the project/getV2 response
- Extracts `data.name`, `data.screen_list[*].ticket_list[*]`
- Reads `screen_name`, `desc`, `sale_flag.display_name` for each ticket
- Returns `(project_name, vector<vector<string>>)` where each inner vector is `{ticket_name, status}`

#### Status Color Mapping (lines 536-543)
```
已售罄 -> Red
已停售 -> Red
不可售 -> Red
未开售 -> Cyan
暂时售罄 -> Yellow
预售中 -> Green
```

#### Key Design Decisions
- **Single API only**: Uses GET `project/getV2` which returns project-level info with text-based status detection
- **Text matching**: Relies on `sale_flag.display_name` string values like "预售中" to determine availability
- **Batch script trigger**: When target ticket goes "预售中", runs an external `.bat`/`.sh`/`.ps1` file
- **No concurrency**: Everything runs on the main thread with `sleep_for()` between polls
- **Terminal-centric**: Heavy use of ANSI escape codes for colored TUI output

---

## 3. HSR_Land_Stock_Monitor.py Deep Analysis

### Architecture

The Python monitor (`extra/HSR_Land_Stock_Monitor.py`) is a multi-target, concurrent ticket stock checker with push notifications.

#### Config Class (lines 15-42)
- All configuration via static class attributes
- `TARGETS`: List of dicts with `screen_id`, `sku_id`, `label` for multi-target monitoring
- `BARK_ENABLED`, `BARK_SERVER`, `BARK_KEY`: Bark push notification settings
- `STOCK_CHECK_URL`: POST `https://show.bilibili.com/api/ticket/stock/check`

#### Monitor Class (lines 53-264)
- Two daemon threads: `show_time()` for status bar, `run_monitor()` for stock polling
- `self.stop` is a `threading.Event` for clean shutdown
- `self.print_lock` is a `threading.Lock` for thread-safe console output
- `self.last_stock_status` tracks per-target stock state for diff detection
- `self.request_count` accumulates total API calls for status bar display

#### HTTP Layer
- Uses `requests.Session` with SSL verification disabled

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fsender/BiliTicketMonitor](https://github.com/fsender/BiliTicketMonitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
