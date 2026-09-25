---
trigger: always_on
description: This file contains the complete policy for `vllm.cpp`. It is the only file that
---

# AGENTS.md: the rules

This file contains the complete policy for `vllm.cpp`. It is the only file that
every agent loads automatically, so every rule lives here. Files under
`.agents/` are task guides. They explain how to do a specific job. They cannot
add or weaken a rule in this file.

The project mirrors vLLM in C++ without PyTorch or a ggml dependency. vLLM
defines the reference behavior and the performance target.

## Start here

1. Run `scripts/agent-start.py`. Pass `--intent operator|helper|read-only` and
   `--row <ID>` when you know them. Otherwise, relay its welcome and ask what
   work is intended. Follow the printed action, then run the command again.
2. Declare a role. Use `scripts/agent-role.py claim operator` for a multi-step
   integration campaign. Use `claim helper --row <ID>` for one scoped task.
   Use `claim read-only` for inspection. The operator claim records the current
   worktree as a coordinator. Another coordinator does not block the claim.
   Add `--headless` only when the developer explicitly says the run is
   unattended. Never infer this setting.
3. Run `scripts/now.py` to get the live position. Read `.agents/NOW.md` for the
   operator's current gate and next actions. The command output is derived.
   The file is authored and fits on one screen.
4. Read only the claimed row, its spec, its evidence, and the task guide for the
   current job.
5. Run `scripts/agent-preflight.sh` before you edit a file.

Never infer a role, host, permission, or developer preference. Resolve `.env`
and `.agents/developer-preferences.md` from the shared checkout. Ask only for
the one value that the current gate needs. If a value is unavailable, leave its
gate `PENDING`. Never convert a missing value into an assumption. Preferences
control operations only. They cannot reduce a correctness, evidence,
attribution, or testing obligation.

**Create both files on first use.** Neither is tracked, so a fresh checkout has
neither, and `scripts/agent-start.py` reports the absence and routes you to ask.
Ask the developer for the one value the current gate needs. Record an
environment value with `scripts/agent-onboard.py --env-set KEY=VALUE`, which
refuses any key `.env.example` does not declare. Record a preference by copying
`.agents/developer-preferences.example.md` and editing the one entry. Leave
every key you did not ask about empty, because empty means unavailable and its
gate stays `PENDING`. A host name, a share path, or a checkout path written in
a repository document is another developer's resolved value. It is never a
default, and reading one instead of asking is the failure this rule names.

## History is git

The project has no state log. Git is the history, and the history must agree
with the tree.

| Question | Command |
|---|---|
| Did this row already land? | `git log --oneline --grep '<ROW-ID>'` |
| When did this symbol change? | `git log -S'<symbol>' --oneline -- <path>` |
| What happened to this file? | `git log --follow --oneline -- <path>` |
| What is on main that I lack? | `git log --oneline HEAD..origin/main` |
| Why is this line like this? | `git log -L '<start>,<end>:<path>'` |
| What did that commit change? | `git show --stat <sha>` |

The roadmap row states the current position of a row. Git and the row's spec
state how it got there. Before you conclude anything about past work, read the
spec and run `git log -S`. Do not derive the history again.

## Every change starts from an issue

**Do not start work without an open canonical local issue.** Issue authority is
one tracked file under `.agents/issues/`; GitHub is an optional mirror. Create a
local issue before claiming a row or writing code:

```sh
python3 scripts/agent-issue.py create \
  --title "..." --kind bug --problem "..." --row ROW-ID
```

Import an existing GitHub issue with `agent-issue.py import-github N`. Imported
text is quoted as historical evidence; it does not override the local record.
Use `update`, `close`, and the explicit `mirror` command for later changes.
Local validation and writes happen before any remote write.

A row-owned issue lives at
`.agents/issues/<ROW-ID>/ISSUE-GH-<number>.md` or
`.agents/issues/<ROW-ID>/ISSUE-LOCAL-<ULID>.md`; its `Row:` field equals the
directory. A rowless issue lives under `_owed`, and exactly one spec lists its
stable local ID under `## Owed`. Assigning a row moves the file and removes that
spec reference in the same change. `_intake` contains migration-only unavailable
archive residue. It is not valid ownership for new work or a full record.

`scripts/agent-issue-index.py --refresh` validates the local files and renders
the untracked `.agents/issue-index.generated.md` convenience view without a
network call. `scripts/check-agent-record.py` validates every canonical issue
and every issue reference in changed files and commit bodies offline. The four
reference forms are `ISSUE-GH-<number>`, `ISSUE-LOCAL-<ULID>`, `#<number>`, and
`issues/<number>`.

A bug that you find during other work still needs a local issue. Filing the
issue does not defer the fix. File it, fix it in the same flow, reference it in
the commit, and close it. The person who found the bug has the context to fix
it. Traceability is the goal, not another round trip.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mudler/vllm.cpp](https://github.com/mudler/vllm.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
