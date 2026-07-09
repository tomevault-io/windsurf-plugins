---
trigger: always_on
description: RULE NUMBER 1 (NEVER EVER EVER FORGET THIS RULE!!!): YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION FROM ME OR A DIRECT COMMAND FROM ME. EVEN A NEW FILE THAT YOU YOURSELF CREATED, SUCH AS A TEST CODE FILE. YOU HAVE A HORRIBLE TRACK RECORD OF DELETING CRITICALLY IMPORTANT FILES OR OTHERWISE THROWING AWAY TONS OF EXPENSIVE WORK THAT I THEN NEED TO PAY TO REPRODUCE. AS A RESULT, YOU HAVE PERMANENTLY LOST ANY AND ALL RIGHTS TO DETERMINE THAT A FILE OR FOLDER SHOULD BE DELETED. YOU
---

RULE NUMBER 1 (NEVER EVER EVER FORGET THIS RULE!!!): YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION FROM ME OR A DIRECT COMMAND FROM ME. EVEN A NEW FILE THAT YOU YOURSELF CREATED, SUCH AS A TEST CODE FILE. YOU HAVE A HORRIBLE TRACK RECORD OF DELETING CRITICALLY IMPORTANT FILES OR OTHERWISE THROWING AWAY TONS OF EXPENSIVE WORK THAT I THEN NEED TO PAY TO REPRODUCE. AS A RESULT, YOU HAVE PERMANENTLY LOST ANY AND ALL RIGHTS TO DETERMINE THAT A FILE OR FOLDER SHOULD BE DELETED. YOU MUST **ALWAYS** ASK AND *RECEIVE* CLEAR, WRITTEN PERMISSION FROM ME BEFORE EVER EVEN THINKING OF DELETING A FILE OR FOLDER OF ANY KIND!!!

### IRREVERSIBLE GIT & FILESYSTEM ACTIONS — DO-NOT-EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. “I think it’s safe” is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it—if anything remains ambiguous, refuse and escalate.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

We only use **bun** in this project, NEVER `npm`, `yarn`, or `pnpm`. We target the latest **Node.js** version for both local development and Vercel deployments, and we do **not** care about compatibility with older Node versions. Dependencies are managed **exclusively** via `package.json` + `bun.lock`; do **not** introduce `package-lock.json`, `yarn.lock`, or any other lockfiles.

### IRREVERSIBLE GIT & FILESYSTEM ACTIONS — DO-NOT-EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. “I think it’s safe” is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it—if anything remains ambiguous, refuse and escalate.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

### Code Editing Discipline

NEVER run a script that processes/changes code files in this repo, EVER. No “code mods” you just invented, no giant regex-based `sed` one-liners, no auto-refactor scripts that touch large parts of the tree.

That sort of brittle, regex-based stuff is always a huge disaster and creates far more problems than it ever solves.

* If many changes are needed but they’re **mechanical**, use several subagents in parallel to make the edits, but still apply them **manually** and review diffs.
* If changes are **subtle or complex**, you must methodically do them yourself, carefully, file by file.

---

### Backwards Compatibility & File Sprawl

We do **not** care about backwards compatibility right now—we’re early, we want the cleanest possible architecture with **zero tech debt**:

* Do **not** create “compatibility shims”.
* Do **not** keep old APIs around just in case. Migrate callers and delete the old API (subject to the no-deletion rule for files; code removal inside a file is fine).

We must AVOID uncontrolled proliferation of code files:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/cmaes_explainer](https://github.com/Dicklesworthstone/cmaes_explainer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
