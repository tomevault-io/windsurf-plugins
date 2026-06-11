---
trigger: always_on
description: **Repo**: `/Users/josh/Developer/alfred/gitx`
---

# git+ Alfred Workflow — Project Context

**Repo**: `/Users/josh/Developer/alfred/gitx`
**GitHub**: https://github.com/jangelsb/git-plus-alfred-workflow

A fully customizable git & zsh interface for Alfred (macOS launcher). It's an Alfred Script Filter: on every keystroke Alfred calls `git_filtering_internal.py` and renders the returned JSON as a menu.

---

## Key Files

| File | Role |
|---|---|
| `git_filtering_internal.py` | Main Alfred Script Filter entry point — parses query, builds result items, outputs JSON |
| `definitions.py` | All data models: `Command`, `CommandType`, `Location`, `Modifier`, `ResultItem`, `TextViewAction`, `TokenizationResult` |
| `actions.yaml` | The default command tree (status, fetch, pull, push, commit, checkout, create, rebase, delete, etc.) |
| `docs.md` | Full YAML field documentation and examples |
| `functions.sh` | Zsh helpers auto-sourced before every command run (`git_stash_checkout_pull`, `view_hunk`, `process_hunk`, `diff_command`, `rebase_multiple_branches_onto`, etc.) |
| `location_arg_parser.py` | Parses a location/repo from an alfred arg |
| `tv_script.py` / `tv_script.sh` | Handles the TextView (in-Alfred text preview) lifecycle |

---

## Architecture: How it Works

### Navigation via autocomplete string
The Alfred query string IS the navigation state. Each `ResultItem` has `autocomplete` set to `"reponame command subcommand "` (trailing space). Selecting an item rewrites the query → script reruns → deeper level shown. No persistent state between calls.

### `tokenize(query, locations, commands)`
Longest-prefix matches the location title, then greedily matches command titles left-to-right. Returns `TokenizationResult(location, commands[], unfinished_query)`.

### `process_commands_recursively()`
When matched command has subcommands, injects them and re-tokenizes at `level+1`. Enables deep paths like `status → modified → modified files → process each hunk`.

### `process_action(action, param, title)`
Template substitution with `shlex.quote` escaping. Replaces `[input]`, `[input_snake_case]`, `[title]`, `[parent]`, `[parent~n]`.

### `construct_full_command(action, location)`
Wraps action as `cd '/path/to/repo';\n<action>`. Resolves `[reload~n]` → Alfred autocomplete path for reloading after command execution.

### `subtitle_for_command()`
If `subtitle_command` is set, actually runs a shell command to produce live subtitle text (e.g. count of staged files, current branch).

---

## CommandType Enum

| Type | Behavior |
|---|---|
| `NO_ACTION` | Shows info, Enter does nothing (e.g. `status`) |
| `SINGLE_ACTION` | Press Enter → runs immediately |
| `NEEDS_PARAM` | Waits for typed `[input]` |
| `NEEDS_SELECTION` | Waits for pick from list (`values`/`values_command`) |
| `INLINE` | Programmatic item generation |

---

## YAML Field Reference (actions.yaml)

```yaml
- title: string               # menu item label / navigation text
  icon: filename.png          # optional, see icons/ folder
  subtitle: string            # static subtitle
  subtitle_command: |         # zsh command whose stdout = subtitle (live data)
    ...
  command: |                  # zsh to run on Enter; supports placeholders
    ...
  values: [a, b, c]           # static selection list
  values_command: |           # dynamic selection list (each line = item)
    ...
  should_use_values_as_inline_commands: bool   # show list items at current menu level
  should_trim_values: bool    # trim whitespace from values (default: true)
  should_use_smart_sort: bool # enable Alfred smart sort (default: false)
  subcommands: [...]          # nested command tree
  mods:                       # modifier key alternate actions
    - mod: cmd|alt|ctrl|fn|shift|cmd+alt
      subtitle: string
      command: |
        ...
  textview_action:            # preview command output in Alfred text view
    command: |
      ...
    mods: [...]               # if omitted, inherits parent mods
  quicklookurl: string        # URL for shift-preview
```

### Dynamic Placeholders

| Placeholder | Meaning |
|---|---|
| `[input]` | User-typed text (shell-escaped) |
| `[input_snake_case]` | Input with spaces → underscores |
| `[input_new_lines]` | Input via TextView (multiline) |
| `[title]` | The current item's title |
| `[parent]` | Immediate parent command title |
| `[parent~n]` | n levels up in command path |
| `[reload]` | After command: reload at current menu level (must be echoed) |
| `[reload~n]` | After command: reload n levels up (must be echoed) |
| `[tv_reload]` | Reload inside TextView |
| `[view in alfred]` | Open repo in Alfred file browser |

### Available Icons
`info.png`, `view.png`, `down.small.png`, `down.big.png`, `up.small.png`, `up.big.png`, `fork.png`, `globe.png`, `tag.png`, `hash.png`, `pencil.png`, `create.png`, `rebase.png`, `revert.png`, `pick.png`, `copy.png`, `trash.png`, `back.png`, `back.line.png`, `open.png`, `search.png`, `folder.png`, `list.png`, `check.png`, `plus.png`, `minus.png`, `action.png`, `icon.png`

---

## functions.sh Helpers (auto-sourced)

- `git_stash_checkout_pull [branch]` — stash → checkout → pull; `--track-remote` for remote branches
- `run_in_terminal <cmd>` — opens Terminal.app to run command interactively

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jangelsb/git-plus-alfred-workflow](https://github.com/jangelsb/git-plus-alfred-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
