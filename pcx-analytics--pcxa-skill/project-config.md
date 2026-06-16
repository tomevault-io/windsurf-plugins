---
trigger: always_on
description: PCXA construction intelligence platform CLI. Search/read files, manage tags and folders, manage activities/steps/progress/dependencies, manage form templates/fields/submissions, manage resources/timesheets/cost-codes/budgets, manage entity links between objects, and chat with the project's AI assistant. Use when the user asks about project files, documents, tasks, activities, forms, resources, timesheets, entity links, project management, or wants to send messages to the AI chatbot.
---


# PCXA CLI

**Tool:** `pcxa`

All commands output JSON by default. Use `-f table` for human-readable. Use `--dry-run` on write operations. Run `pcxa <command> --help` for full options.

When installed as a Claude Code plugin, `pcxa` is provided by the plugin `bin/` directory. For direct terminal use outside Claude Code, ask the user to install it once with:
```
pipx install git+https://github.com/PCX-Analytics/pcxa-skill.git
```
Then `pcxa update` self-upgrades from GitHub. The CLI prints a one-line notice to stderr (max once per 24h) when a newer release is available.


## Setup & Authentication

Always start by running `pcxa whoami` to see the current state. The output tells you whether the user is authenticated and whether a project is set. Only run setup steps that are actually missing.

### Step 1 — Authentication (drive it yourself, don't punt to a terminal)

If `whoami` says "No profiles configured" or a command fails with "Profile not found":

```bash
pcxa login --no-setup
```

The `--no-setup` flag is required when you (the agent) drive login: it skips the interactive company/project picker that reads stdin, which would otherwise hang. The CLI prints two lines immediately:

```
Opening browser to authenticate...
  If your browser does not open automatically, visit:
  https://www.pcxa.app/auth/cli-auth?port=PORT&state=STATE
```

**Read those lines from stdout, then surface the URL to the user as a clickable markdown link.** The CLI tries `webbrowser.open()` but that's usually a no-op in WSL/headless — the user opening the link manually is the normal path. They sign in (MFA/SSO supported), the page redirects to a localhost callback, the CLI captures tokens, the command exits.

Run with a generous bash timeout (e.g., 180s) since the user may take a moment to sign in. The CLI's own `--timeout` defaults to 120s; pass `--timeout 300` if you want longer.

If browser login isn't viable (rare — only if the host can't reach `pcxa.app`), fall back to password login: `pcxa setup -u USER_EMAIL` (prompts for password — only works if the user can run it themselves).

### Step 2 — Pick a project

After login (or if `whoami` shows `Project: not set`), drive the project picker yourself:

```bash
pcxa projects -f table       # list all (company_id, project_id) the user has access to
```

Show that list to the user, ask which project they want, then:

```bash
# When the conversation/CWD is inside a repo that maps 1:1 to a PCXA project,
# pin it locally so future runs in this repo are auto-scoped:
pcxa set-project PROJECT_ID --company COMPANY_ID --local

# Otherwise, set the global default for this user:
pcxa set-project PROJECT_ID --company COMPANY_ID
```

Always pass `--company` when picking — `--local` writes a `.pcxa` file in CWD; without `--local`, the choice is saved in the global profile.

### How resolution works

Project scope resolves in this order: **repo `.pcxa` file** > **active profile default**. `.pcxa` is committed (no secrets — just `{ "company": 4, "project": 10, "user": "alice@example.com" }`). Different repos can pin different accounts via the `user` field; the CLI matches it against profile usernames in the active credentials file.

Credentials resolve **folder-first**: a `.pcxa-credentials.json` found by walking up from CWD is used for both reads and writes (token refresh included); otherwise the global `~/.pcxa/credentials.json` is used. So `pcxa login` from inside a repo writes that repo's own `.pcxa-credentials.json` (at the git root, gitignored) and can't clobber another repo's tokens — pass `--global` to write the shared global file instead. Pre-0.3 global creds at `~/.file_explorer/config.json` are auto-migrated to `~/.pcxa/credentials.json` on first run.

State to surface to the user on the first turn: `whoami` shows `Active profile`, `User`, `Company`, `Project` (with `(from .pcxa)` annotation when applicable), `Creds:`, and `Repo pin:`. If you ran setup steps, echo the resulting scope back ("Operating on project Acme Tower (4)") so the user can correct you before any writes.

## Project Metadata

```bash
pcxa project get                                              # view project details
pcxa project members                                          # list members (name → user ID)
pcxa project members --search "John"                          # search by name/username
pcxa project update --name "New Name"                         # update name
pcxa project update --description "..." --scope-statement "..." # set description & scope
pcxa project update --code "T-FAB1" --industry "Construction" # set code & industry
pcxa project update --start-date 2025-12-03 --end-date 2026-08-10
pcxa project update --rollup-method equal                     # equal|duration|cost|labor
pcxa project update --progress-input-method percentage        # status|percentage
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PCX-Analytics/pcxa-skill](https://github.com/PCX-Analytics/pcxa-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
