---
trigger: always_on
description: Run a WCAG accessibility audit (configurable version and level — default 2.2 AA) using static analysis + axe-core + keyboard + NVDA + VoiceOver + structure + forms on any web app. Guides the user through each step — open a browser, navigate to the state you want tested, say ready. Works with any framework or stack.
---


# Accessibility Audit Skill

A guided WCAG 2.2 AA accessibility audit. The user opens a browser, navigates to the exact page or state they want tested (including logging in, opening dialogs, triggering specific states), and says ready. Claude then runs the selected tests and reports findings. Multiple pages and states can be audited in one session — findings accumulate into a single report.

Works with any web framework or tech stack. Auto-detects project structure where possible; asks when it cannot.

---

## Skill Invocation

```
/a11y-audit [label] [flags]
```

The label (optional) is a short name for what's being audited — used in the report and screenshots. If omitted, the short URL (e.g. `myapp.com/dashboard`) is used automatically once you navigate there.

**Examples:**
```
/a11y-audit --check                            # verify environment and get a suggested command
/a11y-audit --wizard                           # guided setup — Claude walks you through everything
/a11y-audit                                    # URL will be used as the label automatically
/a11y-audit "sign-up form"                     # label provided
/a11y-audit "settings page" --code             # axe scan only
/a11y-audit "contact form" --fix               # all tests + auto-fix
/a11y-audit "navigation menu" --visual         # all tests + visual review per fix
/a11y-audit "profile page" --static --code     # source analysis + axe
/a11y-audit "main nav" --report                # all tests + CSV export
```

---

## Flags

### Setup flags

| Flag | What it does |
|---|---|
| `--install` | Installation guide — asks where to install the skill (current project or global), then outputs the exact commands to run. Exits after printing commands — no audit is started. |
| `--check` | Environment check — verifies Playwright MCP is connected, detects your project setup, and suggests the right command to start your first audit. Run this after installing the skill. |
| `--wizard` | Interactive setup — Claude asks what you want to audit, explains each test, and walks you through the whole process in plain language before anything runs. Ideal for first-time users or when you're not sure which flags to use. |

### Scope flags

| Flag | What it does |
|---|---|
| `--url <url>` | Open the browser directly to this URL and run tests immediately — no manual navigation required. If no label is provided, it is derived from the URL automatically. Subsequent states in the session still use manual navigation. |
| `--wcag <target>` | WCAG version and level to target. Default: `2.2-AA`. Accepted values: `2.0-A`, `2.0-AA`, `2.0-AAA`, `2.1-A`, `2.1-AA`, `2.1-AAA`, `2.2-A`, `2.2-AA`, `2.2-AAA`. Affects the axe scan tag set and is recorded in the audit log and CSV. |

### Test flags

| Flag | What it runs | Requires |
|---|---|---|
| *(none)* | All four browser tests: axe + keyboard + structure + forms | Playwright MCP |
| `--static` | ESLint a11y plugin scan of source files — no browser needed | None |
| `--code` | Axe automated scan only | Playwright MCP |
| `--keyboard` | Keyboard navigation check only | Playwright MCP |
| `--nvda` | NVDA screen reader check — always opt-in, never a default. Windows only. | Playwright MCP + NVDA |
| `--voiceover` | VoiceOver screen reader check — always opt-in, never a default. macOS only. | Playwright MCP + VoiceOver |
| `--structure` | Heading, table, and landmark structure check only | Playwright MCP |
| `--forms` | Form label, required field, and validation check only | Playwright MCP |

### Action flags

| Flag | Description |
|---|---|
| `--fix` | After reporting, automatically fix all found violations. Source code must be accessible in the working directory. |
| `--visual` | Like `--fix`, with a per-file before/after comparison in Playwright — approve or reject each change. Implies `--fix`. |

### Output flags

| Flag | Description |
|---|---|
| `--report` | Generate or update a CSV report in `a11y-screenshots/` with all findings from this session. |
| `--fresh-report` | Like `--report`, but deletes any existing CSV for this label before writing — starts a clean report rather than appending. |
| `--plan` | Generate a Markdown remediation plan — a shareable document with prioritised findings, recommended fixes, and next steps. Saved alongside the CSV in `a11y-screenshots/`. |

Test flags can be combined freely. `--static` works without a browser and can be combined with browser tests. `--fix` and `--visual` assume source code is in the working directory — do not use them when auditing a site you cannot modify.

---

## Implementation

### Phase 1: Setup

#### Step 0I: Installation *(runs when `--install` is passed — exits after printing commands, does not start an audit)*

When the user passes `--install`, do not start an audit. Instead, guide them through installation.

Note the current working directory. Then ask:

> **Where would you like to install the a11y-audit skill?**
>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kylane/a11y-audit](https://github.com/kylane/a11y-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
