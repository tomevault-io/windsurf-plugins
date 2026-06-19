---
trigger: always_on
description: Use when automating the user's already-logged-in Chrome or Edge session — reusing their cookies, SSO, or 2FA — instead of a fresh browser. Primary tool is dev-browser (`dev-browser --connect`); Playwright CLI (`playwright-cli attach`) is the fallback. Triggers on `dev-browser`, `playwright-cli`, CDP attach, or "attach to my open/logged-in browser".
---


# Attaching to a Browser

Use this skill when browser automation must run inside the user's real, already-logged-in
browser session — reusing their cookies, SSO, and 2FA — rather than a fresh automated browser.

Two tools attach to a browser the user already controls. They each run their OWN background daemon
and open their OWN CDP connection, so pick ONE per task — you cannot connect with one tool and drive
with the other.

| Tool | Use it for | Operating model |
|---|---|---|
| **dev-browser** (`dev-browser --connect`) | **Primary.** The most stable connect to the logged-in DEFAULT profile — it auto-discovers that profile's debug endpoint itself, the step that most often breaks by hand. | One-shot JS scripts (Playwright Page API) piped via heredoc. |
| **playwright-cli** (`playwright-cli attach`) | Fallback. A stateful named session driven by discrete subcommands; also ships a dedicated-debug-profile launcher. | `attach` once, then `tab-list` / `snapshot` / `click` … |

**Why dev-browser is the more stable connection.** Attaching to the logged-in DEFAULT profile means
resolving that profile's `DevToolsActivePort` (line 1 = a RANDOM high port, line 2 =
`/devtools/browser/<GUID>`) and connecting to that exact GUID URL. Doing this by hand is the most
failure-prone part of the playwright-cli path, and channel attach (`--cdp=chrome`) resolves only a
bare path that modern Chrome rejects. dev-browser reads `DevToolsActivePort` and builds the GUID URL
itself (Chrome, Canary, Chromium, Brave), falling back to scanning ports 9222–9229 — so the fragile
manual step disappears.

The safety rules at the bottom apply to BOTH tools.

---

## Primary: dev-browser --connect

### Install

```bash
npm install -g dev-browser
dev-browser install   # one-time: downloads Playwright + Chromium
```

The binary is `dev-browser`. Scripts run in a sandboxed QuickJS runtime (NOT Node.js): no
`require`/`import`, `fs`, `process`, or `fetch`. A background daemon starts automatically and keeps
named pages and CDP connections alive across script runs.

### Enable remote debugging (user action — do not bypass)

In the target Chrome/Edge, open `chrome://inspect/#remote-debugging` and enable **Allow remote
debugging for this browser instance**; keep the logged-in browser open. This is a human step every
time — ask for it, never script it. (Alternatively the user launches Chrome with
`--remote-debugging-port=9222`.) If `--connect` reports it cannot auto-discover Chrome, the
browser-side setup in the playwright-cli "Troubleshooting" section still governs — on Chrome 136+ the
default profile may also need a `--remote-allow-origins=*` relaunch.

### Connect and verify the login

`--connect` with no URL auto-discovers the running Chrome. List the tabs FIRST and confirm a
logged-in signal before any real work:

```bash
dev-browser --connect <<'EOF'
const tabs = await browser.listPages();
console.log(JSON.stringify(tabs, null, 2));
EOF
```

Then attach a page and snapshot it. A visible "로그인 / Sign in / 회원가입" button or a fresh
new-tab page means you hit the WRONG (empty) profile — stop and have the user confirm the logged-in
window is the one with remote debugging on. Only proceed once a logged-in signal (account menu, the
user's name, a my-page URL) is present.

```bash
dev-browser --connect <<'EOF'
// Attach to an existing tab by its id from listPages(), or open a named page.
const page = await browser.getPage("work");
await page.goto("https://example.com", { waitUntil: "domcontentloaded" });
const snap = await page.snapshotForAI();   // AI-friendly accessibility snapshot
console.log(snap.full);
EOF
```

### Operate the page

Pages are full [Playwright Page objects](https://playwright.dev/docs/api/class-page) — `goto`,
`click`, `fill`, `locator`, `evaluate`, `screenshot`. Three ways to inspect and act:

- `page.snapshotForAI({ depth, timeout })` → `{ full, incremental? }` — accessibility tree for
  picking targets. This is the "look before you act" step (see Safety rule 2).
- `page.domCua.*` — DOM-id tier: `getVisibleDom()` lists interactive elements as `node_id=N`;
  `click` / `scroll` act by node id; `type` / `keypress` act on the focused element.
- `page.cua.*` — vision tier: `screenshot()` returns a JPEG whose pixels map 1:1 to CSS coordinates;
  `click` / `move` / `scroll` / `type` act at those coordinates.

Named pages persist across script runs (the daemon keeps them), so you can navigate in one script and
act in the next. `browser.newPage()` is anonymous and cleaned up when the script exits. File helpers
(`saveScreenshot`, `writeFile`, `readFile`) are restricted to `~/.dev-browser/tmp/`.

Optional — pre-approve the command so Claude Code does not prompt each run (safe: scripts are
sandboxed). Add to `.claude/settings.json` (project) or `~/.claude/settings.json` (global):

```json
{ "permissions": { "allow": ["Bash(dev-browser *)"] } }
```

Full command + script API surface: [reference/dev-browser.md](reference/dev-browser.md).

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cskwork/attach-to-browser-skill](https://github.com/cskwork/attach-to-browser-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
