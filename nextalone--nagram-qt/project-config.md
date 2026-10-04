---
trigger: always_on
description: This guide defines repository-wide instructions for coding agents working with the Telegram Desktop codebase.
---

# Agent Guide for Telegram Desktop

This guide defines repository-wide instructions for coding agents working with the Telegram Desktop codebase.

## AI Tasks

In this repository "task" is a specific term. It always means one work record in
the sibling `ai-tdesktop` repository, never a `TODO` comment, a checklist item,
or a unit of work invented during the current conversation. "The tasks", "the
queue", "the board", "what's most pressing", and a bare task slug all refer to
that queue.

- The queue lives in `../ai-tdesktop`, a sibling of this checkout, with one
  record per directory under `tasks/YYYY/MM/DD/<slug>/`. A task id is that dated
  path, for example `2026/07/18/fix-community-forward`, and it is the only
  durable link between a source commit and its task. Linked slot worktrees live
  in `../ai-tdesktop-worktrees`. Read `../ai-tdesktop/AGENTS.md` before doing
  anything inside that repository.
- Each record holds `task.md` — one `# ` title line, then one self-contained
  paragraph that is the task's description wherever it is summarized — and
  `state.yaml` with `status`, `type`, `depends_on`, `claimed_by`, and dates.
  Open statuses are `todo`, `in-progress`, `blocked`, and `split-required`;
  `approved` is completed history, and `split.yaml` / `superseded.yaml` mark
  retired records.
- To browse the queue, read those files directly. Do not run `python3 ai.py`: it
  is a full-screen browser for the user's own terminal and refuses to run in a
  pipe. A compact open listing:

```bash
cd ../ai-tdesktop && for state in tasks/*/*/*/*/state.yaml; do
  status=$(sed -n 's/^status: //p' "$state")
  case "$status" in todo|in-progress|blocked|split-required)
    dir=$(dirname "$state")
    printf '%-14s %-70s %s\n' "$status" "${dir#tasks/}" \
      "$(sed -n '1s/^# //p' "$dir/task.md")";;
  esac
done | sort
```

- When ranking open work by urgency, use `in-progress`, then `blocked`, then
  `split-required`, then ready `todo` (no `claimed_by`, and every `depends_on`
  already `approved`), then `todo` still waiting on a dependency. Say who owns
  claimed work. This checkout's tag is the `Telegram/build/ai-machine-tag` value
  plus the checkout folder name, such as `macbook-tdesktop`, and work claimed by
  another checkout is never taken over without an explicit human reassignment.
- Browsing is read-only. Listing, summarizing, comparing, or recommending tasks
  never edits `state.yaml`, publishes a lifecycle commit, or starts
  implementation, no matter how small the task looks.
- Acting on an existing queue task goes through the workflow skills instead of
  by hand: `perform-task <slug or full id>` starts or resumes and performs
  exactly one known task, and `continue` drains eligible shared work. Source
  commits owned by a task use the three-line form described under `## Commits`.
- A request the user makes in conversation is ordinary work: do it directly in
  this checkout. Do not write it to the ignored `../ai-tdesktop/inbox/inbox.md`,
  and do not run `process-inbox` or `continue` over it, unless the user asks for
  that. The queue's planning, review and evidence campaign costs hours, so
  spending it on a request the user expected you to just do is a real error, not
  thoroughness.
- You may *offer* the queue when the work genuinely earns it: a large or
  ambiguous change, one whose correctness needs a real testing campaign, or one
  that is safety-, security- or data-safety-critical. Make the offer in one
  line, say why, and route it only after the user approves. Absent approval,
  implement it directly and say what you skipped.
- Never guess between similarly named tasks. Report the matching full ids and
  let the user choose.

## Working from Codex on Windows + WSL

This checkout may be opened in Codex Desktop through the Windows UNC path `\\wsl.localhost\{distro}\home\{user}\Telegram\tdesktop`, while the real Linux path is `/home/{user}/Telegram/tdesktop`. Treat it as a WSL/Linux checkout first, not as a native Windows checkout.

- Prefer running repository-aware commands through WSL:

```powershell
wsl.exe -d {distro} --cd /home/{user}/Telegram/tdesktop -- <command>
```

- PowerShell can read and write files through the UNC path, but native Windows tools may see different ownership, path, executable, or line-ending behavior than Linux tools.
- Git from PowerShell over `\\wsl.localhost\...` can fail with `detected dubious ownership`. Use WSL Git instead. Do not change global Git `safe.directory` settings unless the user explicitly asks for that.
- Keep path styles matched to the shell. Use `/home/{user}/Telegram/tdesktop/...` with WSL commands, and quoted `\\wsl.localhost\{distro}\home\{user}\Telegram\tdesktop\...` paths with native Windows commands. Avoid passing UNC paths to Linux tools or Linux paths to native Windows tools unless the tool explicitly supports them.
- If a command behaves strangely from the PowerShell UNC working directory, retry the same command through `wsl.exe -d {distro} --cd /home/{user}/Telegram/tdesktop -- ...` before concluding the repository or command is broken.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NextAlone/Nagram-Qt](https://github.com/NextAlone/Nagram-Qt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
