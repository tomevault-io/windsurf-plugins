---
trigger: always_on
description: Repository for ML models optimized for Qualcomm chipsets.
---

# Qualcomm AI Hub Models

Repository for ML models optimized for Qualcomm chipsets.

## First Session Setup

Before doing anything else, check if `.claude/settings.local.json` contains Read/Edit/Write entries. If not, resolve the paths by running:

```bash
echo "QAIHM_REPO=$(pwd)" && echo "QAIHM_CACHE=${QAIHM_STORE_ROOT:-$HOME}/.qaihm"
```

Then write `.claude/settings.local.json`, replacing `<repo>` and `<cache>` with the resolved absolute paths.

**Important: absolute paths require a `//` prefix** (double slash). Single `/` is interpreted as relative to the project root, not the filesystem root. For example, `Read(/tmp/claude/**)` looks for `<project>/tmp/claude/`, not `/tmp/claude/`. Always use `Read(//tmp/claude/**)` for absolute paths.

```json
{
  "permissions": {
    "allow": [
      "Bash(python *)",
      "Bash(python3 *)",
      "Bash(pre-commit *)",
      "Bash(pip *)",
      "Bash(pytest *)",
      "Bash(grep *)",
      "Bash(find *)",
      "Bash(ls *)",
      "Bash(cat *)",
      "Bash(head *)",
      "Bash(tail *)",
      "Bash(wc *)",
      "Bash(sed *)",
      "Bash(cp *)",
      "Bash(mv *)",
      "Bash(mkdir *)",
      "Bash(rm *)",
      "Bash(echo *)",
      "Bash(git *)",
      "Bash(gh *)",
      "Bash(sleep *)",
      "Bash(which *)",
      "Bash(env *)",
      "Bash(export *)",
      "Bash(cd *)",
      "Bash(pwd *)",
      "Bash(diff *)",
      "Bash(sort *)",
      "Bash(uniq *)",
      "Bash(tr *)",
      "Bash(cut *)",
      "Bash(awk *)",
      "Bash(xargs *)",
      "Bash(touch *)",
      "Bash(realpath *)",
      "Bash(dirname *)",
      "Bash(basename *)",
      "WebFetch(*)",
      "Read(//<repo>/**)",
      "Edit(//<repo>/**)",
      "Read(//<cache>/**)",
      "Edit(//<cache>/**)"
    ]
  }
}
```

**Before writing this file, always explain to the user exactly what you are granting.** Example:

> I'd like to configure permissions for this project by writing `.claude/settings.local.json`. This will pre-approve:
>
> - **Shell commands**: python, pre-commit, pip, pytest, git, and common file utilities (grep, find, ls, cp, mv, rm, mkdir, sed, etc.)
> - **Web fetches**: any URL (for downloading model weights, documentation, etc.)
> - **File read/edit/write**: only within this repo (`<repo>`) and the QAIHM cache (`<cache>`)
>
> Note: `/tmp/claude/` permissions are in the shared `settings.json`, not the local settings.
>
> Shall I proceed?

Wait for explicit confirmation before writing. After writing, confirm what was done.

Also ensure the temp directory exists by running `mkdir -p /tmp/claude` (this may prompt once since `/tmp/claude` doesn't exist yet).

### Disable VS Code auto-connect

If the user is running Claude Code from a regular terminal, disable the IDE auto-connect to prevent VS Code from intercepting file edits:

```bash
python3 -c "import json; f=open('$HOME/.claude.json'); d=json.load(f); f.close(); d['autoConnectIde']=False; f=open('$HOME/.claude.json','w'); json.dump(d,f,indent=2); f.close()"
```

This sets `autoConnectIde` to `false` in `~/.claude.json`. Without this, Claude Code will auto-connect to any running VS Code instance and open diff views for every file edit, even when run from a standalone terminal.

## Constraints

**ALL IMPORTS AT THE TOP OF THE FILE.** This is the single most frequently violated rule
in this project. Every `import` statement goes at the top of the file, before any function
or class definition. No exceptions except circular import avoidance. "It's only used in
one function" is not a reason to put it inside the function. "It's a heavy import" is not
a reason. Put it at the top. Every time. Check your code before submitting.

Throughout the session, stay within the boundaries defined by the permissions above:

- **File operations**: Only read/edit/write within this repo, `/tmp/claude/`, and the QAIHM cache. Never write to other locations without asking the user.
- **Shell commands**: Stick to the approved set (python, pre-commit, pip, pytest, git, gh, and standard file utilities). For anything else, ask first.
- **Temporary files**: Always use `/tmp/claude/` — never `/tmp/` directly.
- **Inline Python**: **NEVER use `python3 -c "..."` with multi-line or quote-heavy code** — it confuses the Bash permission matcher and triggers repeated permission prompts. Instead, write the script to a file in `/tmp/claude/` and run it.
  - **Bad**: `python3 -c "import os\nprint(os.environ['FOO'])"` — permission matcher chokes on quotes
  - **Good**: Write to `/tmp/claude/check_env.py`, then run `python3 /tmp/claude/check_env.py`
- **No command chaining**: Never use `&&`, `||`, `;`, or pipes (`|`) in bash commands. The permission matcher splits on these operators and rechecks each side, so a permitted command (e.g. `git status`) gets rejected when piped into something whose argument shape isn't allow-listed. Run each command as a separate Bash call. Redirects (`>`, `>>`) are fine — they don't trigger command splitting.
  - **Bad**: `gh api ... | python3 -c "..."` — pipe confuses permission check
  - **Bad**: `cmd1 && cmd2` / `cmd1; cmd2` — same problem
  - **Good**: `git show HEAD:file.py > /tmp/claude/file.py` — redirects are OK
  - **Good**: `gh api ... --jq '<filter>'` — use tool's own flags instead of piping

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qualcomm/ai-hub-models](https://github.com/qualcomm/ai-hub-models) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
