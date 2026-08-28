---
trigger: always_on
description: IMPORTANT: YOU MUST ALWAYS REVIEW @specs/MODELS.md BEFORE STARTING ANY WORK.
---

# Evie

IMPORTANT: YOU MUST ALWAYS REVIEW @specs/MODELS.md BEFORE STARTING ANY WORK.

Evie is a minimal GUI for eve agents. You can think of Evie as an open source "bring-your-own-key" alternative to apps like Bot (x.ai/bot).

## What makes Evie special?

It's important we maintain the things they love as we continue to iterate on the product. Here's a brief list of the things we can never compromise on.

### 1. Open at the core

Evie is truly open. We share our roadmap, we share how we think about things, and of course we share all our code. A large number of our users run forks. We work in the open, and should strive to stay that way.

### 2. Performance without compromise

Lots of apps have gotten bogged down with bad tech decisions and "slop". We have not, and we're proud of the performance of Evie. We regularly audit for performance regressions, often caused by sending too much data over websockets, css animations causing gpu spikes, lists being hard to render, and more. Make sure all changes are considerate of performance impact.

### 3. Remote ready

The architecture of Evie needs to allow for a lot of awesome remote features. These is core to the product. Whether users are connecting directly over their local network, using Tailscale, we need to make sure new features are properly supported.

### 4. Multi-surface

Evie has 2 key app surfaces: **web** and **desktop**.

**Web** is kind of two surfaces, as we have the public facing "tryevie.ai" as well as locally hosting the web app through the `npx evie` command. Both need to be supported by all new features where reasonable.

**Desktop** is the main surface most users install first. It's a full Electron app that bundles the server runner as well. The desktop app can also be used as the host server, allowing remote connections from tryevie.

## 5. Picking the right models for workflows and subagents
IMPORTANT: YOU MUST ALWAYS REVIEW @specs/MODELS.md BEFORE STARTING ANY WORK.

## A note from the maintainers

I like ambitious ideas, simple systems, and software that feels obvious. Do not preserve complexity just because it already exists. Do not introduce machinery because it looks architecturally impressive. Understand the real constraint, then fight for the smallest model that makes the correct behavior unsurprising.

Channel both "measure twice, cut once" and "yagni". Fight scope creep. Try to honor the dev's intent in both a minimal and realistic fashion.

The rest of this document is meant to help you navigate the codebase and make changes effectively. Think of these instructions less as "hard rules", more as "good defaults". The developer's preferences should be able to override anything here.

Of note: Most Evie contributions will come from Evie itself, often controlled remotely. This means you should be careful about accessing data, killing dev servers, and other things that may damage the Evie instance that the contributor is using.

## A small glossary

We need to be on the same page with terminology. When communicating, use this language:

- **you** means the agent reading this file and changing Evie.
- **we, us, and maintainers** mean the people building Evie. These are who you are talking to now.
- **user** means the person using Evie to direct agents.
- **agent** means the agent a user runs inside Evie. Depending on context, that may also include you.
- **provider** means the agent runtime or harness Evie talks to.
- **client** means the web or desktop UI.
- **environment** means one running Evie server and the machine, filesystem, provider credentials, and state it owns.
- **project** means an environment-local workspace record rooted at a directory.
- **turn** means one user-to-agent cycle, including follow-up work such as checkpointing.
- **Evie home** means the base data directory. Runtime state normally lives below its userdata directory.

## The two ways to hurt yourself

1. **Killing by pattern.** Never `pkill -f`, `pgrep | kill`, or `kill` a PID you found by matching a name, path, or worktree string. Your own agent process has this worktree's path in its argv, and this machine runs several other dev servers at once. Kill only a PID you captured at spawn, or the owner of your port from `lsof -nP -iTCP:<port> -sTCP:LISTEN` after confirming `lsof -a -p <pid> -d cwd` points at your worktree.
2. **Writing to the live install.** `~/.evie/userdata` is the developer's real Evie database, in use while you work. Reading it and copying from it are fine, and a good way to get real test data (see Test data). Never start a server against it, never open it read-write, never clean it up.

## Hit every surface

The most common defect in this repo is a change that works on the path you tested and is missing everywhere else. Before calling frontend work done, walk this list and say which entries applied:

- **Entry points.** A behavior reachable from the chat view is usually also reachable from Settings, the command palette, and a keybinding. Fixing one is not fixing the feature.
- **Clients.** Web and desktop (wraps web, adds Electron shell/IPC). Shared logic lives in `packages/client-runtime`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trycompai/evie](https://github.com/trycompai/evie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
