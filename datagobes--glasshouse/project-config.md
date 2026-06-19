---
trigger: always_on
description: Audit any website against GDPR/ePrivacy and (optionally) turn the findings into a ready-to-file DPA complaint. The skill has two subcommands: `scan` (default — runs a privacy audit and generates a scored presentation + report) and `file` (turns an existing scan into a complaint dossier for a chosen Data Protection Authority).
---


# Glasshouse — privacy audit + DPA complaint builder

Automated privacy audit: scan a website, analyze findings against GDPR/ePrivacy, generate a scored presentation + report. Optionally turn the scan into a ready-to-file complaint dossier.

**Base directory**: The skill lives at the path shown in the "Base directory for this skill" message injected at invocation time. All `cd` commands and file references below use `$SKILL_DIR` as shorthand for that path.

## Subcommand dispatch

The skill has two subcommands sharing one slash command. Parse `ARGUMENTS` to route:

| First token | Mode | Example |
|---|---|---|
| (URL) or `scan <url>` | **scan** (default) | `/glasshouse www.example.com`, `/glasshouse scan www.example.com` |
| `file <scan-json-path>` | **file** (build complaint) | `/glasshouse file /tmp/glasshouse-example.com-*.json` |

If the first token is `scan` or `file`, treat it as the mode and use the remaining arguments as the mode's input. Otherwise default to `scan` mode with the whole argument as the URL.

If `ARGUMENTS` is empty, ask: "Do you want to **scan** a website (give a URL) or **file** a complaint from an existing scan (give a scan JSON path)?"

---

# Scan mode

The user provides a URL. Everything else is determined conversationally or uses smart defaults.

## Conversational flow

1. **Extract the URL** — Parse from the user's message. If missing, ask: "What website should I scan?"
2. **Check for context clues** — Look for hints like "for a client", "corporate", "executive summary", or a company name.
3. **Determine settings** — Only ask when ambiguous:

| Setting | Default | When to ask |
|---------|---------|-------------|
| **Theme** | `corporate` | `datagobes` if maintainer is generating their own deck; `corporate` or `dark` otherwise |
| **Format** | Full (up to 15 slides) | Only if user says "quick", "summary", or "executive" |
| **Branding** | None | The `datagobes` theme adds the maintainer's `>_ datagobes.dev` signature; `corporate` and `dark` carry no branding |

4. **Start immediately** — Don't ask unnecessary questions. The defaults are good.

## Workflow

### Step 1: Prerequisites

Before first use, ensure Playwright is installed:

```bash
cd $SKILL_DIR && npm install 2>/dev/null && npx playwright install firefox 2>/dev/null
```

Skip if `node_modules/playwright` already exists.

### Step 2: Scout the banner

Run a lightweight scout first to detect the consent banner and identify button text — this takes ~10s instead of a full 3-variant scan:

```bash
cd $SKILL_DIR && node scripts/scan.js {URL} --scout
```

**Output**: JSON to stdout with:
- `screenshot` — path to viewport screenshot
- `cmpDetected` — CMP platform name or null
- `bannerDetected` — whether a consent banner was found
- `acceptButtonFound` / `rejectButtonFound` — whether the scanner can click them
- `candidateButtons[]` — all visible buttons in the banner area with `{text, selector, visible}`
- `recommendHints` — `true` if the scanner needs text hints for the full scan
- `suggestedAcceptText` / `suggestedRejectText` — auto-detected button text suggestions

**MANDATORY: Read the scout screenshot using the Read tool.**

Check the screenshot for:
- **Consent banner** — is one visible? Does it match the scout JSON?
- **Button text** — verify the `suggestedAcceptText`/`suggestedRejectText` match what's visible
- **Dark patterns** — asymmetric buttons, hidden reject, pre-checked toggles, colour contrast tricks
- **CMP platform** — identify from visual branding if scanner didn't
- **Cookie wall** — page content blocked behind consent dialog?

### Step 3: Run the full scan

Based on the scout results, run the full scan:

**If `recommendHints` is `false`** (scanner can detect buttons automatically):
```bash
cd $SKILL_DIR && node scripts/scan.js {URL}
```

**If `recommendHints` is `true`** (custom banner needs hints):
Use the button text from the scout results or your visual inspection of the screenshot:
```bash
cd $SKILL_DIR && node scripts/scan.js {URL} --accept-text "Accept all" --reject-text "Reject all"
```

Available hint flags:
- `--accept-text "..."` — Text of the "accept all" button
- `--reject-text "..."` — Text of the "reject all" button
- `--save-text "..."` — Text of a "save preferences" button (used as reject action — saving with toggles off = rejecting)

**Examples by language:**
- Dutch: `--accept-text "Alles accepteren" --save-text "Opslaan"`
- German: `--accept-text "Alle akzeptieren" --reject-text "Alle ablehnen"`
- French: `--accept-text "Tout accepter" --reject-text "Tout refuser"`

- Stdout: JSON file path (last line). Stderr: progress messages.
- **Failures**: Timeout = WAF, bot detection = stealth didn't help, missing Playwright = run step 1.

### Step 4: Verify full scan screenshots

The full scan saves `{base}-ignore-viewport.png` and `{base}-ignore-fullpage.png`. The JSON includes a `screenshots` object with paths.

**MANDATORY: Read the viewport screenshot using the Read tool BEFORE analyzing the JSON.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataGobes/glasshouse](https://github.com/DataGobes/glasshouse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
