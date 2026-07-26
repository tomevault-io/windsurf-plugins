---
trigger: always_on
description: Every skill and agent is a markdown file. No TypeScript, no Python, no package.json. Claude Code interprets the markdown at invocation time. This matters:
---

# Conventions — ADLC Toolkit

## Code is markdown, not code

Every skill and agent is a markdown file. No TypeScript, no Python, no package.json. Claude Code interprets the markdown at invocation time. This matters:

- **No build step**: edits take effect immediately via the symlink install
- **No test runner**: "tests" are dogfooding — invoke the skill on a real REQ and see if it produces the expected artifacts
- **Linting is minimal**: markdown formatting, frontmatter validity, and bash syntax in `!`...`` macros. Nothing else.

**Exception — `tools/`:** the `tools/` directory may contain real executable code (e.g. `tools/delegate/`, a set of Python delegation CLIs with its own `install.sh`). It is exempt from the markdown-only rule and from the symlink-install model — those tools are installed by running their `install.sh`, not via the skills symlink. Each `tools/<name>/` subdirectory carries its own README.

## File and directory naming

- Skill directories: lowercase, single word or hyphenated (`spec`, `bugfix`, `template-drift`)
- Skill files: always `SKILL.md` (uppercase, singular) inside the skill directory
- Agent files: `agents/<agent-name>.md`, hyphenated lowercase
- Templates: `templates/<artifact>-template.md`
- IDs: `REQ-xxx` (zero-padded to 3 digits), `TASK-yyy`, `BUG-zzz`, `LESSON-nnn` — always uppercase prefix, always 3 digits minimum
- Slugs: lowercase kebab-case, ≤6 words, no dates, no bare numbers

## Frontmatter conventions

All artifact types use YAML frontmatter. Dates in ISO format (`YYYY-MM-DD`). Arrays use JSON inline syntax (`tags: [a, b, c]`). Status enum values are lowercase strings.

**Required vs optional** varies per template. Generally: `id`, `title`, `status`, `created` are required; everything else is optional. When adding new fields, prefer additive — do not rename existing fields without a migration plan.

## Ethos injection pattern

Every skill begins with:

```markdown
## Ethos

!`sh .adlc/partials/ethos-include.sh 2>/dev/null || sh ~/.claude/skills/partials/ethos-include.sh`
```

The partial itself emits the canonical fallback chain (consumer-project ETHOS.md first, then toolkit-root, then graceful "No ethos found" message). The two-level fallback at the call site (project `partials/` first, then global `~/.claude/skills/partials/`) ensures the macro still works in consumer projects that haven't re-run `/init` after the toolkit shipped the partial. Never hardcode the ethos body inside a skill — always source the partial.

## Delegation pattern (provider-agnostic)

Skills that delegate bulk reads or drafting to the configured delegate (`adlc-read` /
`adlc-write`) MUST source the shared gate predicate rather than inlining
`command -v adlc-read >/dev/null 2>&1 && …`. The canonical predicate lives in
`partials/delegate-gate.sh` and defines `adlc_delegate_gate_check()` with a 0/1/2 contract
(REQ-522 retired the legacy `kimi-gate.sh` back-compat alias):

```sh
. .adlc/partials/delegate-gate.sh 2>/dev/null || . ~/.claude/skills/partials/delegate-gate.sh
adlc_delegate_gate_check; gate=$?
case $gate in
  0) ;;  # delegated
  1) ;;  # disabled (ADLC_DISABLE_DELEGATE=1, or not opted-in — BR-11)
  2) ;;  # unavailable (adlc-read not on PATH)
esac
```

The reason is exported as `ADLC_DELEGATE_GATE_REASON`. Delegation is **opt-in** (off by
default on fresh installs) — enabled by `delegate.enabled: true` in
`~/.claude/adlc/config.yml`, `ADLC_DELEGATE_ENABLED=1`, or an already-set legacy
`KIMI_API_KEY`/`MOONSHOT_API_KEY` (key continuity, data — REQ-515 BR-11).

Per-step telemetry state crosses the create → gate → invoke → resolve fenced blocks via
the **flag-file sidecar** (`partials/delegate-tools-path.sh`'s `skill-flag.sh mark`/`read`),
never via shell variables, because fenced blocks do not share shell state (REQ-522 BR-4).
The shared resolver `_adlc_emit_step_telemetry <skill> <step>` in
`partials/emit-step-telemetry.sh` reads those marks back and emits one telemetry record.

See `partials/delegate-gate.md` for the full protocol — return-code contract, the
canonical stderr emit templates parameterized by `<skill>` and `<purpose>`, and the BR-4
one-line-per-invocation rule. Per-skill stderr messages and fallback bodies stay inline at
the call site; only the predicate is shared.

## Forge adapter (provider-agnostic PR operations)

Skills that touch the pull-request lifecycle MUST route every PR operation through
the forge adapter (`partials/forge.sh`), never by shelling out to `gh pr` directly.
This makes a project portable between GitHub and Azure DevOps as a config change.
Source the sourceable partial and call the op **in the same fenced block** (the
cross-fence rule above):

```sh
. .adlc/partials/forge.sh 2>/dev/null || . ~/.claude/skills/partials/forge.sh
out=$(adlc_forge_pr_view "$pr" --fields state,url); rc=$?
```

The op set is `adlc_forge_pr_{create,ready,edit,view,list,merge,comment}`. The
GitHub backend (`gh`) is byte-compatible with the previous direct calls; the Azure
DevOps backend uses `az repos`. Provider resolution is per-project `.adlc/config.yml`
`forge.provider` > machine config > `auto` (origin-URL detection; unrecognized host

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atelier-fashion/adlc-toolkit](https://github.com/atelier-fashion/adlc-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
