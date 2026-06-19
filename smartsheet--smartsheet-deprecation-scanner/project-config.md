---
trigger: always_on
description: Use when explicitly invoked via /smartsheet-deprecation-scanner to scan a codebase for usage of deprecated Smartsheet API endpoints and parameters, including both SDK-based and direct HTTP usage.
---


# Smartsheet Deprecation Scanner

## Overview

Scan a codebase for usage of deprecated Smartsheet API endpoints, query parameters, request/response fields, and seat type values. Output a structured markdown report of all findings with file paths, line numbers, code snippets, and migration guidance.

## Trigger

Only activate when explicitly invoked as `/smartsheet-deprecation-scanner`. Do not activate automatically.

---

## Step 0 — Check for Changelog Updates

**Before scanning, always do this first.**

Read the `last_changelog_check` date from this file's YAML frontmatter. If today's date is more than 7 days after that date:

1. Fetch `https://developers.smartsheet.com/api/smartsheet/changelog` using WebFetch with the prompt: "List every deprecation entry. For each one extract: (1) the deprecated item (endpoint, parameter, field, or value), (2) the deprecation date, (3) the sunset date, and (4) the replacement or recommended action."
2. Compare the results against the deprecation categories already documented in this skill (Categories 1–8 and any existing Pending Review entries).
3. For any item in the changelog that is **not already covered** in this skill file:
   - Append it to the `## Pending Review` section at the bottom of this file using the format described there.
   - Update the `last_changelog_check` date in the YAML frontmatter to today's date.
   - Use the Edit tool to write both changes to this skill file.
   - **Alert the user before proceeding with the scan:**

     > ⚠️ **Smartsheet changelog update detected.** New deprecation entries have been added to the `## Pending Review` section of the skill file (`~/.claude/skills/smartsheet-deprecation-scanner/SKILL.md`). Please review and promote or discard them before relying on this scan as complete.
     >
     > New entries found:
     > - [list each new item with its deprecation date and sunset date]
     >
     > Continuing scan with currently confirmed deprecations only…

4. If no new items are found, still update `last_changelog_check` to today's date and continue.
5. If the changelog fetch fails, note it at the top of the report and continue with the existing deprecation data:

   > ⚠️ **Changelog check skipped** — could not reach developers.smartsheet.com. Scan uses deprecation data last verified on {last_changelog_check}.

If `last_changelog_check` is within 7 days of today, skip the fetch and proceed directly to Step 1.

---

## Pending Review

<!-- New deprecations found by the auto-update check are appended here.
     Each entry should be reviewed and either promoted into the appropriate
     Category section above (with full grep patterns and migration guidance)
     or removed if not applicable.
     Format per entry:
     ### [PENDING] {short description}
     - **Deprecated:** {date}
     - **Sunset:** {date or TBD}
     - **Item:** {endpoint / parameter / field / value}
     - **Replacement:** {replacement or "none"}
     - **Source:** changelog entry dated {date}
     - **Status:** Awaiting review
-->

---

## Scan Targets

### Languages & SDKs to cover

- **Java** — `smartsheet-java-sdk`
- **C#** — `smartsheet-csharp-sdk`
- **Python** — `smartsheet-python-sdk`
- **Node.js** — `smartsheet-javascript-sdk`
- **Ruby** — `smartsheet-ruby-sdk`
- **Go, PHP, Scala, Swift, Kotlin, and any other language** — no official Smartsheet SDK exists for these; scan for direct HTTP usage only. All URL path patterns apply equally regardless of language.
- **Direct HTTP** — `axios`, `fetch`, `HttpClient`, `requests`, `RestTemplate`, `HTTParty`, `Net::HTTP`, `URLConnection`, `WebClient`, `HttpURLConnection`, `curl`, `http.Get`, `http.Post`, `GuzzleHttp`, `file_get_contents`, etc.

---

## Deprecation Reference

### Category 1 — Deprecated Endpoints (Sharing)

Asset-specific sharing endpoints are deprecated. **Deprecated:** 2025-08-04. **Sunset:** 2026-06-03.

**Replacement:** Use unified `/shares` endpoints with `assetType` and `assetId` query parameters, using `PATCH` (not `PUT`) for updates.

> **HTTP method change alert:** `PUT /.../shares/{shareId}` → `PATCH /shares/{shareId}`. This is both a URL change AND a method change. Flag it explicitly in the report — some HTTP clients will silently send the wrong method. Mark these findings with **⚠ HTTP METHOD CHANGE** in the occurrence table.

| Deprecated | Replacement |
|---|---|
| `GET /sheets/{id}/shares` | `GET /shares?assetType=sheet&assetId={id}` |
| `GET /reports/{id}/shares` | `GET /shares?assetType=report&assetId={id}` |
| `GET /sights/{id}/shares` | `GET /shares?assetType=sight&assetId={id}` |
| `GET /workspaces/{id}/shares` | `GET /shares?assetType=workspace&assetId={id}` |
| `GET /sheets/{id}/shares/{shareId}` | `GET /shares/{shareId}?assetType=sheet&assetId={id}` |
| `GET /reports/{id}/shares/{shareId}` | `GET /shares/{shareId}?assetType=report&assetId={id}` |
| `GET /sights/{id}/shares/{shareId}` | `GET /shares/{shareId}?assetType=sight&assetId={id}` |
| `GET /workspaces/{id}/shares/{shareId}` | `GET /shares/{shareId}?assetType=workspace&assetId={id}` |
| `POST /sheets/{id}/shares` | `POST /shares?assetType=sheet&assetId={id}` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smartsheet/smartsheet-deprecation-scanner](https://github.com/smartsheet/smartsheet-deprecation-scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
