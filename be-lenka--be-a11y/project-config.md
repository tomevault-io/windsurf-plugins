---
trigger: always_on
description: This file is the stable, machine-oriented contract for driving be-a11y from
---

# AGENTS.md — machine contract for `@belenkadev/be-a11y`

This file is the stable, machine-oriented contract for driving be-a11y from
scripts, CI, and coding agents. Humans: see `README.md`.

be-a11y audits HTML / template projects (and live URLs) for WCAG 2.1 / EAA
accessibility issues. It runs 29 rules producing 36 issue types, and emits a
structured JSON report.

---

## Invocation

```bash
be-a11y <dir|file|url> [report.json|report.html] [--json] [--list-rules] [--help]
# or, from a checkout:
node index.js <dir|file|url> [report.json|report.html] [options]
```

- **Positional 1** — a directory (scanned recursively), a single file (scanned
  regardless of extension), or an `http(s)://` URL.
- **Positional 2** (optional) — path to write the report to (written **even
  when clean**). The format is inferred from the extension: `.html` / `.htm`
  (case-insensitive) → self-contained HTML page; anything else → JSON
  (schema v2).
- Flags are position-independent. Unknown flags or a 3rd positional are a usage
  error (exit 2).

### Flags

| Flag | Effect |
|---|---|
| `--json` | Print the full JSON report (schema v2) to **stdout** and nothing else on stdout. |
| `--list-rules` | Print all rules + metadata as JSON to stdout, then exit 0. |
| `--help`, `-h` | Print usage to stdout, exit 0. |

### Exit codes (stable)

| Code | Meaning |
|---|---|
| `0` | No accessibility issues found. |
| `1` | Accessibility issues found. |
| `2` | Usage or environment error (no input, bad path, fetch failure / HTTP non-2xx, report-write failure, unknown flag). |

### stdout vs stderr

- **stdout** — the report only: the human report (incl. the `🚨 Accessibility
  Issues Found:` banner and summary), the `✅ No accessibility issues found!`
  line, or the JSON (`--json` / `--list-rules`).
- **stderr** — diagnostics only: usage/errors, the config warning, per-rule crash
  notices (`⚠️ Rule "<id>" crashed …`), and the `📦 Results exported…` info line.

**Recommended for agents:** run with `--json` and parse stdout. Do not grep the
human output.

---

## JSON report (schema v2)

Identical document for `--json` (stdout) and the report file.

```jsonc
{
  "schemaVersion": 2,
  "tool": { "name": "@belenkadev/be-a11y", "version": "3.0.0" },
  "target": "./public",                 // the scanned dir/file/URL, or null
  "timestamp": "2026-01-01T00:00:00.000Z",
  "summary": {
    "filesScanned": 12,
    "total": 9,
    "errors": 6,                        // count of severity:"error" issues
    "warnings": 3,                       // count of severity:"warning" issues
    "byType": { "missing-alt": 4, "…": 5 }  // keys sorted
  },
  "issues": [                            // sorted by (file, line, type)
    {
      "file": "public/index.html",       // legacy fields (v1-compatible)
      "line": 42,
      "type": "missing-alt",
      "message": "<img> is missing an alt attribute",
      "ruleId": "alt-attributes",        // enrichment (v2)
      "severity": "error",               // "error" | "warning"
      "wcag": ["1.1.1"],                  // Success Criteria; [] = best practice
      "hint": "Add a descriptive alt attribute, or alt=\"\" if decorative.",
      "snippet": "<img src=\"hero.jpg\">" // trimmed source line, ≤120 chars
      // "column": 5                      // present only if the rule provided one
    }
  ]
}
```

Field notes:
- `issues[]` always contains the four legacy keys (`file, line, type, message`)
  plus the enrichment keys (`ruleId, severity, wcag, hint, snippet`).
- An unknown emitted type enriches to `ruleId:null, severity:"error", wcag:[],
  hint:null` and prints one stderr warning.
- `severity` is the field to branch on. `errors` block CI (exit 1); treat
  `warnings` per your policy.

---

## HTML report

A report path ending in `.html` / `.htm` writes a **self-contained HTML page**
instead of JSON: inline CSS/JS, system fonts, zero external requests, light/dark
via `prefers-color-scheme`, readable with JavaScript disabled. It renders the
same data as the JSON report (verdict, per-file totals table, issues grouped by
rule with severity/WCAG badges, hints, and source snippets) plus client-side
severity filtering, text search, and expand/collapse.

- **Agents should parse the JSON, not the HTML.** The HTML page is presentation
  for humans (CI artifacts, sharing); its markup is not a stable contract. Use
  `--json` or a `.json` report path for machine consumption.
- The rendered page passes be-a11y itself (`analyzeContent(html, "report.html")`
  → `[]`) — enforced by the test suite (dogfood).
- All report data is HTML-escaped into text nodes; hostile content in scanned
  files (script tags, template syntax, quotes) cannot break or script the page.

---

## `--list-rules`

```jsonc
{
  "schemaVersion": 2,
  "rules": [
    {
      "id": "alt-attributes",           // config key
      "description": "Images must have appropriate alt attributes",
      "defaultEnabled": true,
      "types": [
        { "type": "missing-alt", "severity": "error", "wcag": ["1.1.1"],
          "hint": "…", "label": "Missing ALT", "emoji": "🖼️" }
      ]
    }
  ]
}
```

29 rules, 36 types. `id` is the config key; each `types[].type` is what appears
as `issues[].type` in a report.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [be-lenka/be-a11y](https://github.com/be-lenka/be-a11y) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
