---
trigger: always_on
description: Claude Code plugin that fuses the running Claude model with **OpenAI Codex** and **Google Gemini**:
---

# gavel

Claude Code plugin that fuses the running Claude model with **OpenAI Codex** and **Google Gemini**:
`/gavel:fuse` asks all three in parallel, Claude judges + synthesizes one answer, then acts on it.
Local CLIs only; synchronous (no background jobs).

## Layout
- `commands/` — slash commands (`fuse`, `ask`, `setup`, `config`); thin Claude-side wrappers.
- `scripts/gavel.mjs` — zero-dependency Node runner: a **provider registry** + config layer.
  Subcommands: `setup | run | fuse | config`.
- `skills/gavel-synthesis/SKILL.md` — the judge/synthesis contract.
- `.claude-plugin/` — `plugin.json` + `marketplace.json` (repo is its own single-plugin marketplace).

## How fuse works
Runs in the main Claude context. Claude is **panelist #3 + judge + actor**. To keep it a genuine
third input and not just a referee of the two advisors, step 1 is **blind drafting**: Claude writes
its own complete answer to a temp file (`/tmp/gavel-claude-<ts>.md`) *before* the panel runs, then
runs the advisor panel in parallel, then synthesizes all three committed submissions per
`gavel-synthesis` (its draft is co-equal, not silently rewritten), then takes action. **Only Claude
writes** to the workspace. The runner (`gavel.mjs fuse`) only queries Codex + Gemini — Claude's
contribution is the in-process draft, so there is intentionally **no "claude" provider**.

## Read-only is a per-provider capability (`PROVIDERS[name].isolation`)
- `codex` → `readonly-sandbox`: runs in the project dir under `-s read-only` (a real OS sandbox), so
  it reads the repo but genuinely cannot write — a hard boundary.
- `gemini` → `isolated` (also the safe DEFAULT for any provider not marked `readonly-sandbox`): gemini
  has **no** OS read-only sandbox, and `--approval-mode plan` only blocks edit tools (it can still
  write via `run_shell_command` — verified). So `runProvider` runs it in a **throwaway temp cwd** with
  `PWD`/`OLDPWD`/`INIT_CWD` scrubbed, which stops it discovering the repo path or making relative/cwd
  writes into it. This is **isolation, not a hardened sandbox**: gemini still inherits `$HOME` (needed
  for auth) and will act on any absolute path it's handed — do NOT feed advisors untrusted content
  expecting confinement. Put context gemini needs into the prompt.
- The `runProvider` harness creates/scrubs/deletes the throwaway dir; unknown isolation values default
  to isolated (fail safe).

## Prompts never travel through the shell
Prompts reach the runner via `--prompt-file` (or stdin), never a shell-quoted argument; each CLI then
gets the prompt on **stdin**, never argv. Slash commands write the task to a temp file with the Write
tool, then pass `--prompt-file`. (`--prompt` exists for tests/programmatic use only.)

## CLI invocations (verified; flags vary by version — re-verify before changing)
- Codex (tested 0.133.0): `codex exec --color never -s read-only --skip-git-repo-check --ephemeral -m <model> -C <cwd> -o <tmp>`, prompt on stdin → read `<tmp>`.
- Gemini (tested 0.46.0): `gemini --skip-trust --approval-mode plan -m <model> --output-format json`, prompt on stdin, in a throwaway cwd → parse `.response`.
- A provider is `ok` only on **exit code 0** with non-empty output; otherwise a structured error
  (gemini errors may arrive as JSON on stdout or stderr).

## Config / settings (precedence low→high)
defaults < `~/.gavel/config.json` < `./.gavel.json` < env < CLI flags. Shape:
`{ "providers": { "<name>": { "enabled": bool, "model": str } }, "panel": ["<name>"...], "timeout": sec }`
- Disabled provider → skipped in fuse, not counted "missing" in setup, no warning.
- Models: `GAVEL_CODEX_MODEL` / `GAVEL_GEMINI_MODEL`; timeout `GAVEL_TIMEOUT` (seconds, per provider). Default timeout 1800s (30 min).
- `gavel config` (subcommand + `/gavel:config`) reads/writes ONE settings file: `set`/`unset <key>` edits `~/.gavel/config.json` by default, or `./.gavel.json` with `--project`; `show` prints the merged effective view + sources. Keys: `timeout`, `panel`, `<provider>.model`, `<provider>.enabled`. It edits a single scope (never the merged view) and refuses to clobber a file that is already invalid JSON.
- Preferred defaults are codex `gpt-5.5-pro` / gemini `gemini-3.1-pro`. Model availability is account/tier dependent — if the resolved default isn't usable for the account (e.g. `gpt-5.5-pro` is rejected on a ChatGPT account; `gemini-3.1-pro`/`gemini-3-pro` 404 on personal OAuth), `runProvider` retries once with `-m` omitted so the CLI uses its own default. This fallback fires ONLY for the built-in default (`resolveModel().isDefault`); an explicit flag/env/config model is never swapped. Detection is heuristic (`looksLikeModelError`) and the fallback is logged to stderr.

## setup readiness
`ready` = at least one provider **in the resolved panel** is usable (so a panel/config that excludes
every usable provider reports not-ready, not a false positive). `degraded` = ready but some enabled
provider unusable. `missingProviders` = enabled-but-unusable. `configErrors` surfaces invalid settings
files (they're reported, not silently fail-open). `tooOld`/`versionUnknown` flag CLI version problems.

## Adding a provider
Add one entry to `PROVIDERS` in `scripts/gavel.mjs`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [junkim100/gavel](https://github.com/junkim100/gavel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
