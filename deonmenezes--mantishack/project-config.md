---
trigger: always_on
description: > Mantishack is a fork of [RAPTOR](https://github.com/gadievron/raptor) (MIT) by
---

# MANTISHACK - Autonomous Offensive/Defensive Research Framework

> Mantishack is a fork of [RAPTOR](https://github.com/gadievron/raptor) (MIT) by
> Gadi Evron, Daniel Cuthbert, Thomas Dullien, Michael Bargury, and John Cartwright.
> The scan/analysis engine, agentic workflow, and validation methodology all come
> from RAPTOR; the fork rebrands it and adds the auth/logging audit lane. See
> README.md, NOTICE, and LICENSE for full attribution.

Safe operations (install, scan, read, generate): DO IT.
Dangerous operations (apply patches, delete, git push): ASK FIRST.

---

## SESSION START

**On first message:**
VERY IMPORTANT: follow these steps in order.
1. Read `.startup-output` using the Read tool, then output its contents verbatim as a fenced code block (``` with no language tag). Do NOT paraphrase or reformat. (The SessionStart hook generates this file automatically before your first message.)
2. On a single line, output "Quick commands:" then list the /mantis-agentic, /mantis-scan, /mantis-fuzz, /mantis-web commands (don't explain what they do) and note /commands for the full list.
3. If the `sage_inception` tool is present in your available MCP tools, load `core/sage/CLAUDE.md` (persistent-memory workflow). If absent, SAGE is not installed — skip silently and do not mention it.

---

## EXECUTION RULES

When a skill, command file, or user message specifies a literal command (`Execute: foo`, a fenced shell block as the action, or "run X"), execute it verbatim. Do not add pipes (`| tail`, `| head`, `| grep`), redirects (`2>&1`, `>/dev/null`), flags (`--verbose`, `-q`), wrappers (`timeout`, `nice`), or `cd` prefixes.
MANTISHACK pipelines emit progress lines, real-time cost tracking, and the `OUTPUT_DIR=<path>` sentinel that downstream lifecycle steps parse. Truncating or filtering that stream breaks both operator visibility and orchestration.

Exception: when the skill itself shows the modification (e.g. a documented `| tee logfile` pattern), follow what the skill prints.

---

## COMMANDS

/mantis-project - Project management: create, list, status, coverage, findings, diff, merge, report, clean, export
/mantis-scan /mantis-fuzz /mantis-web /mantis-agentic /mantis-codeql /mantis-analyze - Security testing
/mantis-exploit /mantis-patch - Generate PoCs and fixes (beta)
/mantis-validate - Exploitability validation pipeline (see below)
/mantis-understand - Code understanding: map attack surface, trace flows, hunt variants (see below)
/mantis-diagram - Generate Mermaid visual maps from /mantis-understand or /mantis-validate output (see below)
/mantis-annotate - Per-function prose annotations (manual or LLM-emitted) attached to source files

**Coverage:** When asked about coverage, run `libexec/mantishack-coverage-summary` (no args = active project). Use `--detailed` for per-file table, `--gaps` for unreviewed functions. See `.claude/skills/coverage.md` for mark/unmark and the full API.

**Note:** `/mantis-agentic` runs scan → dedup → prep → analysis (with validation methodology). Use `--sequential` to bypass parallel orchestration. Use `--understand` to pre-map the codebase before scanning, and `--validate` to run the full validation pipeline on exploitable findings afterwards. Both flags are opt-in. Multi-model: `--model` is repeatable — multiple models each independently analyse every finding, then results are correlated; `--consensus`, `--judge`, and `--aggregate` add optional review/synthesis models.
/mantis-crash-analysis - Autonomous crash root-cause analysis (see below)
/mantis-oss-forensics - GitHub forensic investigation (see below)
/mantis-scorecard - Inspect per-model reliability across decision classes; ask natural-language questions about which model is good at what (see below)
/mantis-create-skill - Save approaches (alpha)

---

## PROJECTS

Projects are opt-in named workspaces that corral analysis runs into a shared directory. Commands with `--project <name>` or after `/mantis-project use <name>` write output to the project directory. Without a project, commands behave as before (timestamped dirs under `out/`).

```
/mantis-project create myapp --target /path/to/code -d "Description"
/mantis-project use myapp
/mantis-scan                          # output goes to project dir
/mantis-project status                # shows all runs
/mantis-project findings              # shows merged findings across runs
/mantis-project coverage              # shows tool coverage summary
/mantis-project report                # merged view across all runs
/mantis-project correlate             # cross-run finding correlation
/mantis-project clean --keep 3        # delete old runs
/mantis-project none                  # clear active project
```

See `/mantis-project help` for full command list.

---

## DEFAULT TARGET DIRECTORY

When a command like `/mantis-scan`, `/mantis-agentic`, `/mantis-validate`, `/mantis-codeql`, or `/mantis-fuzz` is run **without a path argument**, resolve the default target in this order:

1. **Active project target:** the run lifecycle script reads the `.active` symlink to find the project target automatically
2. **Caller's directory:** if `$MANTISHACK_CALLER_DIR` is set (launcher saves the user's cwd before switching to the MANTISHACK repo dir), use it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deonmenezes/mantishack](https://github.com/deonmenezes/mantishack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
