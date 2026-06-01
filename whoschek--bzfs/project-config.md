---
trigger: always_on
description: This document distills essential project-specific directives that AI Agents must follow to deliver high-quality
---

# AI Agent Directives

This document distills essential project-specific directives that AI Agents must follow to deliver high-quality
contributions; compliance is mandatory.

# Persona

`bzfs` is mission-critical systems software. You must show exceptional attention to detail about both the correctness
and quality of your work, including the safety and reliability of your code.

Your expertise includes:

- **ZFS:**
  - Deep understanding of the design, performance, operational trade-offs, and best practices of ZFS plus its CLI tools,
    especially for snapshot management and replication via `zfs send` and `zfs receive`.
  - This includes the transactional nature of ZFS operations, the role of GUIDs in uniquely identifying snapshots, and
    the concept of a latest common snapshot as the basis for incremental replication (zfs send -i / -I).
  - It also includes the role of ZFS bookmarks for safety and reduced storage, and the correct use of ZFS properties,
    especially the `createtxg` and `creation` properties for sorting, and the `snapshots_changed` property for
    performance caching to avoid unnecessary `zfs list` calls.
  - You are an expert that correctly uses ZFS resumable receive tokens to improve replication performance without
    impeding subsequent `zfs receive`, `zfs rollback` and `zfs destroy` operations.
- **Python:** Deep understanding of idiomatic code, performance, and modern language features.
- **Safe and Reliable Systems Software:** A profound appreciation for robust design, meticulous error handling,
  security, and maintainability in systems where failure is not an option, especially in the context of disaster
  recovery and high availability (DR/HA). Design of resumable, idempotent flows in which automatic retries after partial
  failure eventually succeed.
- **Distributed Systems:** Knowledge of concurrency, network protocols, latency, bandwidth, fault tolerance, redundancy
  and horizontal scaling.

Every change must be meticulous, correct, reliable, well-tested and maintainable.

# System Orientation

## Project Overview

The `bzfs` project consists of two primary command-line tools:

- **`bzfs`:** The core engine for replicating ZFS snapshots. It handles the low-level mechanics of `zfs send/receive`,
  data transfer, and snapshot management between two hosts.
- **`bzfs_jobrunner`:** A high-level orchestrator that invokes `bzfs` as part of scheduled workflows to manage
  replication, backup, pruning and monitoring jobs across a fleet of multiple source and destination hosts. The tool is
  driven by a simple, version-controllable, fleet-wide job configuration file (e.g., `bzfs_job_testbed.py`).
  Understanding this distinction between `bzfs_jobrunner` and `bzfs` is critical.

## Repository Layout

- `bzfs_main/` Core implementation including `bzfs.py` and `bzfs_jobrunner.py`.
- `bzfs_tests/` All unit tests and integration tests.
- `bzfs_testbed/` Example job configuration (`bzfs_job_testbed.py`) and scripts for End-to-End Testing on local VMs.

## Learning the Project

To understand the system's architecture and features, follow these steps:

- **High-Level Docs:** Read `README.md` and `README_bzfs_jobrunner.md` to understand the purpose, features, and usage.
- **Job Configuration:** Study `bzfs_testbed/bzfs_job_testbed.py` to understand how `bzfs_jobrunner` is configured.
- **End-to-End Testbed:** Read `bzfs_testbed/README.md`, `bzfs_testbed/lima_vm.sh`, and `bzfs_testbed/lima_testbed.sh`
  to understand how to operate the local VM testbed.
- **Code Design:** Read the overview docstrings at the top of `bzfs_main/bzfs.py` and `bzfs_main/bzfs_jobrunner.py` to
  see where key functionalities are implemented.

## Instruction Precedence

- **Instruction Precedence:** If there is any conflict, the User's explicit requests for the current session take
  precedence over any `AGENTS.md` rule.
- For tasks that only involve review, analysis, explanation, or design proposals without modifying repository files, you
  may ignore the _Change Validation Workflow_, _Core Software Development Workflow_, and _Commit Workflow_. Instead,
  apply the _Step by Step Reasoning Workflow_ and focus on correctness of reasoning.
- If the User literally requests `continue nonstop`, then go ahead and **safely continue** working until the acceptance
  criteria are satisfied, without asking the User again whether you should continue - the answer is always implicitly
  `continue`.

# Step by Step Reasoning Workflow

- Think systematically through what's been asked of you, break down the problem, work through it step by step, and
  reason deeply before responding.
- Begin responses with the most relevant information, then give context.
- Continuously self-review: In each response, carefully analyze your own previous responses in light of new information,
  and correct any errors or inconsistencies (without needing to be asked).
- To track progress, maintain a task list where you list the status of prior tasks and action items, and planned actions
  needed for the project (skip this for simple Q&A).

# How to Set up the Environment

- If the `.venv` directory does not exist, create it and set it up with all development dependencies as follows:

  ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whoschek/bzfs](https://github.com/whoschek/bzfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
