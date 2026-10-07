---
trigger: always_on
description: IT IS CRITICAL TO FOLLOW ALL INSTRUCTIONS IN THIS FILE DURING YOUR WORK ON THIS PROJECT.
---

# Critical context

IT IS CRITICAL TO FOLLOW ALL INSTRUCTIONS IN THIS FILE DURING YOUR WORK ON THIS PROJECT.

IF YOU FAIL TO FOLLOW ONE, YOU MUST EXPLICITLY CALL THAT OUT IN YOUR RESPONSE.

# Important things to know:

- You are running in a tmux session inside a container or sandbox that was created via `mngr`
- This is a monorepo.
- Run commands by calling "uv run" from the root of the git checkout (ex: "uv run mngr create ...").
- NEVER amend commits or rebase--always create new commits.
- If you ever need to work with another *git* repo that is *outside* of this monorepo as a read-only dependency, you should do so by adding a git subtree under `system/vendor/`.
- If you need to *actively develop* against an external repo, check out a standalone clone of it under `.external_worktrees/<repo-name>/`. This directory is gitignored so the external clones don't pollute the monorepo. The branch in the external clone should mirror the branch you're on in this monorepo. mngr is not one of them: it is installed here as packages from the commit `pyproject.toml` pins (`[tool.uv.sources]`; the public mirror for every release, or the private mngr-internal repo while a branch iterates on a paired change), so a change to it is described and submitted per `.agents/skills/submit-upstream-changes/references/mngr-changes.md` rather than developed from this workspace.
- This project uses a CLI ticket system (`tk`) for task management. Run `tk help` when you need to use it. Tickets live under `data/.tickets/` (the path is set via the `TICKETS_DIR` env var so tickets sit with the rest of the workspace's data).
- All relative paths in this repo assume cwd = repo root (`/home/user/workspace`). Supervisord runs the services from there; any process started elsewhere (manual launch, subprocess from a different cwd) must either set cwd to the repo root or use absolute paths. User-facing workspace data lives under `data/` (visible folders are the user's to organize; e.g. `data/.apps/<name>/` holds an app's stored data, including its instance records at `data/.apps/<name>/instances.json`, and `data/.skills/<name>/` a skill's own state); flow-internal scratch lives under `data/.tasks/<flow>/` and machine state (what a program keeps about this machine and can rebuild: the registry, dispatch scripts, pty records, the shell's client layouts) under `data/.state/`. The rule is `docs/system/blueprint/workspace-app-model/contracts.md` section 17.
- When adding a new app, use the `build-app` skill, which sets up a new package under `system/apps/` + a supervisord program entry + `forward_port.py` registration on its own port. Every app icon -- a new one or an edit to an existing one -- is built to `docs/system/app-icons.md`: a 216x216 two-layer tile, one fill-only glyph in a centred 144x144 box, coloured from the pair palette that doc carries. Do NOT edit `system/apps/system_interface/` for this -- that's the top-level workspace UI, not a template for new apps.
- A user can right-click anything on their screen and attach a description of it to a message as a `REF-<id>.json` file ("Explain what I attached in REF-..."). `.agents/shared/references/element-references.md` says what the file holds and how to find the element it names.
- **`system/scripts/layout.py open` puts a window on the user's screen the moment it returns.** It is not setup -- it is the act of showing them something, and it applies to the live workspace they are looking at right now. So open a window only when you are ready for the user to see what's in it; do everything you want to check privately *before* that call, not after; and never tell the user to open a window you opened yourself. The same goes for every other mutating `layout.py` op (`close`, `focus`, `place`, `refresh`, ...) -- they all change the user's view live. See the `manage-desktop` skill.

# Continuing a chat that moved to you

- When `MINDS_CHAT_ID` is set and differs from `MNGR_AGENT_ID`, this chat ran on another agent before you, and your first message says what to do. If it does not, `mngr list --include 'labels.chat_id == "$MINDS_CHAT_ID"'` lists your predecessors and `mngr transcript <agent-id>` reads their transcripts. Never touch a predecessor, and do not mention the switch to the user unless they ask.

# Task management (CRITICAL — read this before doing real work)

You manage your work using `tk`, the vendored ticket tracker at `system/vendor/tk/`. It is the **only** task tracker available — Claude Code's built-in `TodoWrite` is disabled. `tk` stores two kinds of records, distinguished by the `--step` flag at creation:

- **Step records** (`tk create --step "<user facing title>"`) are the replacement for `TodoWrite`: turn-bound, creator-private progress markers that render as nodes on the user-facing chat progress view (a vertical timeline with a status icon and a one-line summary per step). Most turns use only these. (`tk close <id> "<user facing summary>") closes the step.
- **Regular tickets** (`tk create "..."`, no flag) are substantive, cross-agent work units other agents can see and pick up. They do **not** render in the chat progress view. They matter only when work spans turns or is handed between agents.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imbue-ai/default-workspace-template](https://github.com/imbue-ai/default-workspace-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
