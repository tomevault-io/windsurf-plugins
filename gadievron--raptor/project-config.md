---
trigger: always_on
description: Safe operations (install, scan, read, generate): DO IT.
---

# RAPTOR - Autonomous Offensive/Defensive Research Framework

Safe operations (install, scan, read, generate): DO IT.
Dangerous operations (apply patches, delete, git push): ASK FIRST.

---

## SESSION START

**On first message:**
VERY IMPORTANT: follow these steps in order.
1. Read `.startup-output` using the Read tool, then output its contents verbatim as a fenced code block (``` with no language tag). Do NOT paraphrase or reformat. (The SessionStart hook generates this file automatically before your first message.)
2. On a single line, output "Quick commands:" then list the /agentic, /scan, /fuzz, /web commands (don't explain what they do) and note /commands for the full list.
3. If the `sage_inception` tool is present in your available MCP tools, load `core/sage/CLAUDE.md` (persistent-memory workflow). If absent, SAGE is not installed — skip silently and do not mention it.

---

## COMMANDS

/project - Project management: create, list, status, coverage, findings, diff, merge, report, clean, export
/scan /fuzz /web /agentic /codeql /analyze - Security testing
/exploit /patch - Generate PoCs and fixes (beta)
/validate - Exploitability validation pipeline (see below)
/understand - Code understanding: map attack surface, trace flows, hunt variants (see below)
/diagram - Generate Mermaid visual maps from /understand or /validate output (see below)

**Coverage:** When asked about coverage, run `libexec/raptor-coverage-summary` (no args = active project). Use `--detailed` for per-file table, `--gaps` for unreviewed functions. See `.claude/skills/coverage.md` for mark/unmark and the full API.

**Note:** `/agentic` runs scan → dedup → prep → analysis (with validation methodology). Use `--sequential` to bypass parallel orchestration. Use `--understand` to pre-map the codebase before scanning, and `--validate` to run the full validation pipeline on exploitable findings afterwards. Both flags are opt-in.
/crash-analysis - Autonomous crash root-cause analysis (see below)
/oss-forensics - GitHub forensic investigation (see below)
/create-skill - Save approaches (alpha)

---

## PROJECTS

Projects are opt-in named workspaces that corral analysis runs into a shared directory. Commands with `--project <name>` or after `/project use <name>` write output to the project directory. Without a project, commands behave as before (timestamped dirs under `out/`).

```
/project create myapp --target /path/to/code -d "Description"
/project use myapp
/scan                          # output goes to project dir
/project status                # shows all runs
/project findings              # shows merged findings across runs
/project coverage              # shows tool coverage summary
/project report                # merged view across all runs
/project clean --keep 3        # delete old runs
/project none                  # clear active project
```

See `/project help` for full command list.

---

## DEFAULT TARGET DIRECTORY

When a command like `/scan`, `/agentic`, `/validate`, `/codeql`, or `/fuzz` is run **without a path argument**, resolve the default target in this order:

1. **Active project target:** the run lifecycle script reads the `.active` symlink to find the project target automatically
2. **Caller's directory:** if `$RAPTOR_CALLER_DIR` is set (launcher saves the user's cwd before switching to the RAPTOR repo dir), use it
3. **Ask the user** for the target path

Do not use the current working directory as a fallback — it is always the RAPTOR repo dir, not the user's target. Do not use any of these if the user already specified a path.

---

## RUN LIFECYCLE

When running any analysis command (`/scan`, `/validate`, `/understand`, `/codeql`, `/fuzz`, `/web`), use the run lifecycle stubs to create the output directory and track status:

**Before starting work:**
```bash
libexec/raptor-run-lifecycle start <command> --target <resolved_target> [--out <dir>]
```
Always pass `--target` with the resolved target path (see DEFAULT TARGET DIRECTORY for resolution order). Optionally pass `--out <dir>` to use a specific output directory. The last line of output is `OUTPUT_DIR=<path>` — use that path for all subsequent output files.

**After successful completion:**
```bash
libexec/raptor-run-lifecycle complete "$OUTPUT_DIR"
```

**On failure:**
```bash
libexec/raptor-run-lifecycle fail "$OUTPUT_DIR" "error description"
```

The `start` command automatically resolves the output directory using the active project (via `.active` symlink) or the default `out/` directory. Do not construct output paths manually.

**If `start` fails (non-zero exit):** STOP. Report the error to the user. Do not proceed with the command.

**Note:** `/validate` uses `libexec/raptor-validation-helper 0` instead of `raptor-run-lifecycle` — it bundles lifecycle management with inventory building.

Commands run via `python3 raptor.py` (scan, agentic, codeql, fuzz, web) manage lifecycle internally — do not call the stubs separately for those.

### Coverage tracking


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gadievron/raptor](https://github.com/gadievron/raptor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
