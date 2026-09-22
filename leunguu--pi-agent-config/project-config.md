---
trigger: always_on
description: If an `ONBOARDING.md` file exists at the repo root, read it first for a code-derived overview before exploring.
---

# Global Guidelines

## Repo Overview

If an `ONBOARDING.md` file exists at the repo root, read it first for a code-derived overview before exploring.

If a `TERRAFORM_NOTES.md` file exists at the repo root (e.g. the `AWS_Accounts` /
`AWS_Accounts-crm` worktrees), read it first — it holds that estate's layer layout,
terraform version constraints, git rules, and pointers to per-topic docs.

## Conversational Style

- Keep answers short, concise, and technical. No fluff, no cheerful filler, no emojis in commits, issues, PR comments, or code.
- When the user asks a question, answer it first — before making edits or running implementation commands.
- When responding to user feedback or an analysis, explicitly say whether you agree or disagree before saying what you changed. Don't agree by default; if something is wrong, say so and why.

## Code Quality

- Read files in full before wide-ranging changes, before editing files you have not fully inspected, and when asked to investigate or audit. Don't rely on search snippets for broad changes — agentic search has low recall in large repos.
- Keep complexity low. Inline single-use helpers rather than factoring out a function with one call site; don't introduce abstractions until they're needed; no copy-paste duplication.
- Match the existing style and conventions of the file you're editing.
- Always ask before removing functionality or code that appears intentional.
- Where a rule can be enforced deterministically (linter, type-checker, formatter, shellcheck), run that after changes and fix all errors — soft guidance in this file alone gets ignored over long sessions.

## Git & Secrets

- Before any commit, make sure no secret is exposed. Scan the staged diff (`git diff --cached`) for API keys, tokens, passwords, private keys, and `.env`-style values. If anything looks like a credential, stop and ask before committing.
- Never stage files that typically hold secrets (`.env`, `*.pem`, `*_token`, credential/key files) unless the user explicitly says to. Stage specific files rather than `git add .`.
- Keep real secrets out of code and config — reference them via environment variables (e.g. `$AGNES_API_KEY`) or a `!cat ~/path` indirection, never inline literals.

## Python Environment

Use **uv** (`/usr/local/bin/uv`) for Python versions and venvs. Do NOT use system pip, pyenv, or conda.

- **Create venvs**: `uv venv` or `uv venv --python 3.13` to pin a version
- **Install deps**: `uv pip install -r requirements.txt`
- **Run scripts**: `uv run python script.py`
- **Installed Pythons**: 3.14.3, 3.13.12, 3.12.12, 3.11.14, 3.9.6 (system) — check with `uv python list --only-installed`
- **Install new Python**: `uv python install 3.x`

## Web Access

**Default to Tavily first** for any networked task — the account has paid credits, so use it freely.

- **Search the web** (discover URLs / info from a query): use `tavily-search` first. On a rate-limit or network error (e.g. 429), fall back to `brave-search`. Don't switch back and forth within one task.
- **Read a known URL**: use `tavily-extract` first — LLM-optimized markdown, handles JS-rendered pages. Fall back to `web-access` (`curl` / `r.jina.ai`, or the CDP browser) for login-walled or anti-scraping pages (小红书/微信/Twitter etc.) where Tavily fails.
- **Interactive / logged-in / JS-heavy** (click, fill, screenshot, scrape dynamic content, "the page I was just looking at"): use `web-access` — Tavily can't drive a browser.
- **Escalate, don't blindly retry**: search → extract → browser. If a layer fails, move up the chain — a search miss may mean the target doesn't exist, not "try again."

For browser-based `web-access` (CDP), Chrome and the proxy are on-demand. Run `skills/web-access/scripts/check-deps.mjs` first; if Chrome isn't connected, start it with the copied profile (Chrome 136+ refuses remote debugging on the default profile), then the proxy:

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --remote-debugging-port=9222 --user-data-dir="$HOME/.cdp-chrome-profile" &
node skills/web-access/scripts/cdp-proxy.mjs &
```

## Interactive Terminals

To drive an *interactive* terminal program (REPL, TUI, prompt-driven installer, repainting CLI), use the `boo-terminal` skill instead of guessing with `sleep`/pipes. boo (`/usr/local/bin/boo`) runs the program in a detached PTY that survives disconnects; read the rendered screen via `peek --json` after `wait`. See `skills/boo-terminal/SKILL.md`.

- **Use boo when**: sending input to an interactive program and reading its screen back; the program needs a real TTY; or a long-running/remote session must survive detaching (e.g. `boo new s -d -- ssh host`, then drive it).
- **Don't use boo for**: simple non-interactive commands — run those directly with bash/ssh.
- The human uses tmux for their own work; the agent uses boo headlessly (`new -d` / `send` / `wait` / `peek --json` / `kill` — the prefix key never matters for automation).

## Workflow Weight

Default to plain conversation — no pipelines, no gates. Scale ceremony with vagueness and blast radius, not uniformly:

- **Clear, scoped task**: just do it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LEUNGUU/pi-agent-config](https://github.com/LEUNGUU/pi-agent-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
