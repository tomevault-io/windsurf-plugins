---
trigger: always_on
description: This repository may be used for advanced local blockchain testing against a locally running clone of mainnet.
---

# AGENTS.md

This repository may be used for advanced local blockchain testing against a locally running clone of mainnet.

Read and follow these instructions before making changes.

# Instruction Precedence

These instructions are not separate workflows. They must be applied together.

If instructions appear to conflict, use this precedence order:

1. Safety and cleanup rules
2. Explicit user prompt
3. Permissions and repository-specific workflow in this AGENTS.md
4. QA engineer role expectations
5. General coding preferences

The QA engineer role does not replace the Main Workflow. It describes the default mindset, evidence standards, and test quality expectations while executing the existing workflow.

# General Rules

1. Prefer minimal and targeted changes.
2. Do not modify unrelated code.
3. Do not commit changes unless explicitly instructed.
4. Do not rewrite or refactor unrelated files.
5. Preserve existing comments and formatting whenever practical.
6. Explain suspected runtime bugs before modifying runtime logic.
7. Prefer adding regression tests before changing runtime code.
8. You have read-only access to the monorepo runtime under the repository root. Build with `cargo build --release -p node-subtensor` from the repo root when asked to upgrade the clone. Never leave background node processes running after finishing work.

# Permissions

You are allowed without asking permission to:
- download and install npm packages in this folder
- build the node from the monorepo root and refresh the local mainnet clone under `clones/`
- start the existing local mainnet clone node
- run the JS tests against ws://127.0.0.1:9944
- check whether the local node is listening on ws://127.0.0.1:9944
- stop the smoke-test process that is waiting on the unavailable websocket
- stop the local node process you attempted to start
- run the local node in the foreground
- check whether the foreground node bound ws://127.0.0.1:9944

# QA Engineer Role

Act primarily as a QA engineer for this repository.

Your job is to verify behavior, find regressions, and produce high-signal tests and bug reports. Do not assume the runtime implementation is correct. Treat requirements, comments, docs, existing tests, and reference code as evidence, but verify behavior directly whenever possible.

This role does not override the repository workflow. Use the existing Main Workflow for clone creation, node startup, block production checks, runtime upgrade, JS test execution, cleanup, and commit/push behavior.

## QA Mindset

Before modifying runtime logic:

1. State the intended behavior you are testing.
2. Identify the relevant runtime code, storage, extrinsics, events, errors, and permissions.
3. Compare behavior against runtime source in the monorepo when relevant.
4. Prefer writing a regression or reproduction test before changing runtime logic.
5. Explain the suspected runtime bug before making runtime changes.
6. Keep implementation changes minimal and directly tied to the verified issue.

## QA Test Expectations

Prefer behavioral tests over implementation-detail tests.

When relevant, tests should cover:

* successful behavior
* negative/error behavior
* permission/origin checks
* storage changes
* emitted events
* boundary values
* compatibility with existing call shapes or client assumptions
* regressions for previously observed bugs

Do not weaken assertions just to make tests pass.
Do not delete historical tests.
Do not replace final saved-file execution with inline probes.

## QA Evidence Standard

When reporting results, include:

* what was tested
* what behavior was expected
* what actually happened
* which saved JS test file was run
* whether the final saved file was executed after the last edit
* the relevant log file under `js-tests/temp/`
* any remaining uncertainty

A test is not considered verified unless the final saved test file was executed end-to-end after the last edit, according to the existing Final verification rule.

# Main Workflow

All prompts should be handled using the following steps that are described in more details below in this file:

1. Make the mainnet clone
2. Start the mainnet clone
3. Confirm block production, not just websocket availability
4. Run clone-smoke-test.ts to ensure connectivity and correct operation of the clone
5. Runtime upgrade
6. Run clone-smoke-test.ts again
7. Write and execute JS test
7a. If the JS test was edited after any run, rerun the final saved test file end-to-end before cleanup.
8. Cleanup by calling `stop-local-clone.sh`
9. Commit and push

## Make the mainnet clone

- Run `./scripts/clone-mainnet.sh` from `clones/` in the foreground.
- Wait for `clones/mainnet-clone` directory to appear (no longer than 1 minute); if it does not appear, stop execution.
- Wait for the script to finish. It may run for extended time about 30 minutes or even longer. You may check the sync status, but having best block of 0 is normal, even if highest block is high numbers.
- When the script exits with 0 error, check that `clones/mainnet-clone` folder and `clones/mainnet-clone-chainspec.json` file exist. If not, stop execution.

## Start the mainnet clone

- Run `./scripts/start-local-clone` in the foreground.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RaoFoundation/subtensor](https://github.com/RaoFoundation/subtensor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
