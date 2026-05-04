---
trigger: always_on
description: task test # Run the full test suite in daemon mode (this is the default, when the user asks to run tests, this is the command)
---

## Build & Test Commands

```bash
# Install a git-ai debug build for local dev on the system so that all git commands will route through it.
# Installs to the same location as real release builds, so it overrides system-wide. It also runs `git-ai install`
# and restarts the daemon to ensure all latest code changes are fully installed and propagated system-wide.
# Use this for trying out changes locally -- do not use any other approaches for runing git-ai locally. They will
# not work, interfere, and break things.
task dev

# Build (only use this for checking that your changes compile)
task build

# Test (use these commands to run the test suite -- these calls are optimized for your system; all flags/args/modes can be combined)
task test # Run the full test suite in daemon mode (this is the default, when the user asks to run tests, this is the command)
task test TEST_FILTER=foo # run specific test
task test NO_CAPTURE=true # Run with Cargo's --no-capture flag
task test EXTRA_TEST_BINARY_ARGS="--ignored" # ignored / exact / other flags
task test CARGO_TEST_ARGS="--lib" # cargo-level flags (rare)

# If the user explicitly asks for tests to be run in another mode (do not run test using these commands unless this test mode is explicitly asked for by the user)
task test:wrapper-daemon
task test:wrapper

# Lint & Format
task lint
task fmt

# Snapshot management (insta crate)
cargo insta review                       # interactively review snapshot changes
cargo insta accept                       # accept all pending snapshots
```

## PR Workflow

Before opening a PR, make sure to run `task lint` and `task fmt` and resolve any formatting/lint issues as they will fail in CI.

When opening a PR, make sure to monitor the ubuntu-based CI jobs first. They are the fastest (roughly 15mins) and if they fail, you should quickly iterate based on those failures and update the PR -- iterating there until those jobs are all green. Additionally, while you're checking on the ubuntu-based jobs, our automated PR review bot, Devin, should have had time to leave feedback. Make sure to read all of Devin's PR review feedback commits and address them. Address them means review, understand, evaluate, and fix if necessary or comment with your thoughts if you don't the feedback is a real issue. Once the lint, fmt, and Ubuntu-based tests have passed and you have addressed all Devin PR review feedback, you can stop monitoring CI for the Mac (~35mins) and Windows (up to 3.5 hours) checks unless the user has explicitly asked for you to wait for those or you're working on a specific OS-based bug.

## Architecture

### Binary dispatch (src/main.rs)

A single binary serves two roles based on `argv[0]`:
- **`argv[0] == "git"`** --> `commands::git_handlers::handle_git()` -- proxies to real git with pre/post hooks per subcommand
- **`argv[0] == "git-ai"`** --> `commands::git_ai_handlers::handle_git_ai()` -- direct subcommands (checkpoint, blame, diff, status, search, etc.)
- **Debug-only shortcut**: When `cfg!(debug_assertions)` and `GIT_AI=git` env var is set, forces git proxy mode regardless of binary name. This is how integration tests invoke the binary as a git proxy without symlinking.

### Core data flow: checkpoint --> working log --> authorship note

1. **Checkpoint**: An AI coding agent calls `git-ai checkpoint <agent>` with hook input (typically JSON via stdin) before AND after it edits a file. The corresponding agent preset (`src/commands/checkpoint_agent/agent_presets.rs`) extracts edited file paths, transcript, and model info. The checkpoint processor diffs the file against HEAD's version or the last-checkpointed value of that file and compute character-level attributions. The combination of pre and post file edit checkpoints is what allows us to know exactly what the AI changed (since we can compare the before and after). There are 3 main types of checkpoints in git-ai:
    * Plain or legacy `human`: only due to legacy, it's still called `human` as it used to mean "human" edited files, but since we migrated to an explicit Human checkpoint (now called `known_human`), this checkpoint represents 'untracked' changes. This is the checkpoint that AI agent presets invoke to take the before edit snapshots. Changes caught by these checkpoints do get explicit attestations in the final authorship notes (they are basically holes in the data) and stats recognize them as untracked. For testing, invoke by calling `git-ai checkpoint human` (for unscoped) or `git-ai checkpoint human /path/to/file` (for scoped).
    * Known human (`known_human`) checkpoints: this is the 'real' Human checkpoint. These are never called by the AI agent presets and are only invoked by our IDE/editor extensions that recognize when a change has actually been made by the human by typing, etc. For testing, invoke via `git-ai checkpoint mock_known_human` (for unscoped) or `git-ai checkpoint mock_known_human /path/to/file` (for scoped).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [git-ai-project/git-ai](https://github.com/git-ai-project/git-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
