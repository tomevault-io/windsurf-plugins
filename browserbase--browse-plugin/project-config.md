---
trigger: always_on
description: Use `browse` as the primary Browserbase command-line interface.
---

# Browse CLI

Use `browse` as the primary Browserbase command-line interface.

It can:

- drive a local or Browserbase-hosted browser session
- inspect pages through accessibility snapshots, screenshots, DOM/text reads, and network capture
- interact with pages by refs, selectors, XPath, keyboard, mouse, files, and viewport controls
- manage Browserbase projects, sessions, contexts, extensions, fetch, and search APIs
- develop, publish, and invoke Browserbase Functions
- browse and scaffold Browserbase templates
- diagnose local or remote browser setup issues
- discover and install Browse.sh catalog skills
- install or refresh this Browse CLI skill

## Setup Check

Verify the CLI exists before relying on it:

```bash
which browse || npm install -g browse
browse --help
```

Install or refresh this skill with:

```bash
browse skills install
```

Use `browse <topic> --help` for exact flags before running unfamiliar commands.

## Browser Target Selection

Browser driver commands auto-start the browse daemon when needed. Choose the browser target per command with flags:

```bash
browse open https://example.com --local
browse open https://example.com --local --headed
browse open https://example.com --remote
browse open https://example.com --remote --verified --proxies
browse open https://example.com --auto-connect
browse open https://example.com --cdp 9222
browse open https://example.com --cdp ws://127.0.0.1:9222/devtools/browser/<id>
```

Use local mode for development, localhost, trusted sites, and fast iteration. Use `--auto-connect` only when the user explicitly wants to attach to an already-running debuggable Chrome session with existing cookies or login state; use `--local` when no debuggable Chrome is available. Use remote mode when Browserbase credentials are available and the site needs hosted browser infrastructure, Verified browser mode, CAPTCHA solving, proxies, or session persistence.

`--local` requires Chrome or Chromium already installed on the machine. In containers, CI, and sandboxes with no browser installed, use `--remote` instead of `--local`. If `--local` fails with "No Chrome or Chromium found" and `BROWSERBASE_API_KEY` is set, switch to `--remote` — do not retry `--local`.

For a Verified and/or proxied remote session, add `--verified` and/or `--proxies` to `--remote` — a single command that keeps the Browserbase session identity, so `browse status` and `browse doctor` report the session ID and live-view URL. `--verified` requires a Browserbase Scale plan. These flags only apply to `--remote` and are sticky for the session's lifetime, like `--headed`. Reach for `browse cloud sessions create` + `--cdp` only when you need session options `open` doesn't expose (region, keep-alive, contexts).

Choose headed/headless and local/remote mode when starting a session. A running session keeps its mode: passing a conflicting flag such as `--headed` to an already-running headless session fails until you run `browse stop --session <name>` or target a different session.

Use named sessions for any non-trivial work, especially when multiple agents or parallel tasks may run at once. Every browser command accepts `--session <name>` (or `-s <name>`); the `BROWSE_SESSION` env var sets the default, and commands without either share the `default` session.

If `BROWSE_SESSION` is already set in the environment, every command already targets that session — do not pass `--session` or invent a new name. An explicit `--session <name>` always overrides `BROWSE_SESSION` for that command, so only pass it to deliberately target a different session.

```bash
browse open https://example.com --session research --local
browse snapshot --session research
```

Remote browser and cloud API commands require:

```bash
export BROWSERBASE_API_KEY=...
```

## Browser Automation Workflow

Start by opening the page, then inspect state, act, and verify.

```bash
browse open https://example.com --session research --local
browse snapshot --session research
browse click @0-5 --session research
browse type "hello" --session research
browse snapshot --session research
browse stop --session research
```

Prefer `browse snapshot` over screenshots for most browser work. It is structured, fast, and returns refs like `@0-5` for reliable element interaction. Use screenshots when visual layout, images, or pixel-level state matter.

Refs are refreshed on every snapshot. After clicks, form submits, navigation, or UI re-renders, take a new snapshot before using another ref.

## Parallel Browser Work

Use a different `--session` value for each independent browser task. Sessions isolate tabs, cookies, refs, and daemon state; parallel tasks that omit `--session` share the `default` session and overwrite each other's active page.

```bash
browse open https://example.com/search-a --session search-a --local
browse open https://example.com/search-b --session search-b --local
browse snapshot --session search-a
browse snapshot --session search-b
```

When a task is complete, stop only that task's session:

```bash
browse stop --session search-a
```

## Core Browser Commands

Navigation:

```bash
browse open <url>
browse reload
browse back
browse forward
browse wait load
browse wait selector "#result"
```

Page state:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [browserbase/browse-plugin](https://github.com/browserbase/browse-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
