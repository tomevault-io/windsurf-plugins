---
trigger: always_on
description: Use when querying Hangzhou Xiaoshan International Airport (HGH/ZSHC) flight departures, arrivals, timetables (including day-of-week schedules), or city autocomplete. Documents the complete set of JSON APIs reverse-engineered from hzairport.com with curl recipes, response schemas, and usage patterns.
---


# Hangzhou Xiaoshan Airport (HGH/ZSHC) API

## Overview

The official Hangzhou airport website (`hzairport.com`) exposes a set of JSON APIs that return real-time flight data and static schedules. All endpoints are **unauthenticated**, accept GET (and POST), and return clean JSON. They were reverse-engineered from the site's own `ajax.js` (located at `/static/front/js/ajax.js`), which defines jQuery AJAX wrappers calling these endpoints server-side.

**Base URL:** `https://www.hzairport.com`

**Architecture:** The frontend is a traditional jQuery server-rendered site (~2018 era). Flight data is loaded asynchronously via synchronous `$.ajax({async:false})` POST calls to `_more.html` endpoints that return paginated JSON. All endpoints also accept GET with query-string parameters — no auth tokens, CSRF, or session cookies required.

---

## When to Use

- User asks about a specific flight from/to Hangzhou (HGH)
- User wants to know today's departures/arrivals at HGH
- User asks "what flights leave Hangzhou at X time" or similar time-based queries
- User asks about flight schedules for a specific weekday (e.g. "Friday 12:00")
- User asks about a flight's status, gate, aircraft type, or airline
- User needs to find valid city/airport names for filtering (autocomplete)

Do NOT use for:
- Other Chinese airports (each has its own site/API)
- Booking or purchasing tickets (status/schedule only)
- Historical flight data (only current-day live data + static schedule)
- Future-date live status (the live endpoints ignore any `date` parameter)

---

### Quick Decision: Which Endpoint?

| Question type | Endpoint | Key detail |
|---------------|----------|------------|
| "What flights leave Hangzhou today at X:XX?" | `/flight/index_more.html` | `md_name` = destination |
| "Flight CA4598 — real-time status?" | `/flight/index_more.html?identity=CA4598` | `operation_type_en` |
| "What flights land in Hangzhou today at X:XX?" | `/flight/arrive_more.html` | ⚠️ `sf_name` = origin (NOT `md_name`) |
| "Does flight X operate on Fridays?" | `/flight/more_time.html?cates=arrive&keywords=X` | `schedule` field: "5"=Friday |
| "What flights operate on day X at time Y?" | `/flight/more_time.html` + manual filter | `schedule` + `arrive`/`takeoff` |
| "Give me the full schedule for route A→B" | `/flight/more_time.html?cates=arrive&keywords=B` | Paginated scan |
| "What are valid city names for filtering?" | `/flight/out_air_more.html?city=X` | Returns `[{title: "城市/机场"}]` |
| "Find a flight number by partial match" | `/flight/out_air_more.html?identity=X` | Returns matching flight numbers |

---

## Endpoints

### 1. Departures — `GET /flight/index_more.html`

**Live departure board for the current day only.** The `date` parameter, if passed, is ignored by the backend.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `p` | int | No | Page number (default 1, 10 items/page, ordered by scheduled time ascending) |
| `identity` | string | No | Filter by flight number (e.g. `CA4598`) |
| `city` | string | No | Filter by destination city in Chinese (e.g. `成都`) |
| `airline` | string | No | Filter by airline name in Chinese (e.g. `国际航空`) |

**Minimal example:**
```bash
curl -s "https://www.hzairport.com/flight/index_more.html?p=1"
```

**Response schema:**
```json
{
  "flag": 1,
  "msg": "OK",
  "data": [
    {
      "flight_identity": "CA4598",
      "airline_name": "国际航空",
      "iata_flight_number": "CA4598",
      "md_name": "成都/双流",
      "md_name_en": "Chengdu/Shuangliu",
      "jhsj_time": "2026-05-19 20:00:00",
      "sjsj_time": null,
      "bgsj_time": null,
      "operation_type": "计划",
      "operation_type_en": "On Schedule",
      "flight_sector_code": "国内",
      "gate_id": "B14",
      "check_indesk_range": "F17-F29",
      "air_craftty": "空客A321(窄体大型)",
      "jt_name": "",
      "jt_name_en": ""
    }
  ],
  "extra": {}
}
```

**Key fields:**

| Field | Type | Description |
|-------|------|-------------|
| `flight_identity` | string | Flight number (primary identifier) |
| `airline_name` | string | Airline in Chinese (see Airline Mapping table) |
| `md_name` / `md_name_en` | string | Destination city/airport (中/EN) |
| `jhsj_time` | string | **Scheduled time** (`YYYY-MM-DD HH:MM:SS`) |
| `sjsj_time` | string\|null | **Actual time** — null until operated |
| `bgsj_time` | string\|null | **Estimated time** (预估时间) — populated when en-route, useful fallback |
| `operation_type_en` | string | `On Schedule` \| `Departed` \| `Delayed` \| `Cancelled` |
| `flight_sector_code` | string | `国内` \| `国际` \| `地区` |
| `gate_id` | string | Boarding gate (often empty for future flights) |
| `check_indesk_range` | string | Check-in counter range |
| `air_craftty` | string | Aircraft type in Chinese |
| `jt_name` / `jt_name_en` | string | Stopover/via city (empty if direct) |

### 2. Arrivals — `GET /flight/arrive_more.html`

**Live arrival board for the current day only.** Same parameter interface as departures.

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [janebooom/hgh-airport-api](https://github.com/janebooom/hgh-airport-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
