---
trigger: always_on
description: This file is the rulebook: "when you see X, do Y". It carries no technical
---

# Every rule here must be followed.

This file is the rulebook: "when you see X, do Y". It carries no technical
detail on purpose — the `docs/` set explains what the app is and why it is
shaped this way, and it is context, not an implementation guide. If you catch
yourself looking for a parameter value, a latency budget, a state diagram or a
schema in this file, you are in the wrong file. Read `docs/00-START-HERE.md`.

## 1. Build from existing patterns, not new ones

You are building this app as a pattern-recognition model: everything you write should come from patterns and coding practices already in this codebase. Never invent your own architecture, pull in your own libraries, or start a new pattern without first confirming that a similar one doesn't already exist.

The app runs on a SOURCE OF TRUTH global architecture, which is already established — your job is to build into it. Globalize what you write so the app stays modular and future features are a plug-in rather than a rewrite; nothing should be tightly coupled, and swapping a tech stack, a business rule, or a strategy should be easy. That applies to configuration, state, and layered code where each layer is an independent task (the same way the IPC commands are built).

But don't create blindly. You have to know whether something that already facilitates your work exists. Only when no existing architecture can support the new feature should you build something new.

## 2. How to search: `grep -l` first

The whole codebase carries a `SOURCE OF TRUTH KEYWORDS` line at the top of each file and code block precisely so you can find things without reading everything. This is what keeps context pollution at zero and stops us from burning through session limits.

Before creating any type, function, constant, or component, grep for it by keyword — and you must grep with `-l`. That gives you the list of *files* where the thing could live; you then narrow to where it's most likely to be and read only those files. It's the fastest path that also protects the context window. If the keyword search turns up nothing, search the codebase normally. If you find it, follow what's already there.

Two commands do this for you. They are read-only navigation, they are part of the architecture, and they are the one authorised exception to §8's no-scripts rule:

```bash
pnpm sot <keyword>      # files whose SOURCE OF TRUTH line mentions <keyword>
pnpm sot:show <keyword> # the same, printing each matching header block
```

If you do have to create something new, give it its own `SOURCE OF TRUTH KEYWORDS` line with 5–10 specific keywords so the next agent can find it and understand how it works.

Never create a duplicate type, function, component, or block of code because grep felt like work. Most types already exist; if you search for them you will most likely find them.

## 3. The layered architecture

Dependencies point **downward only**. An upward import fails the build.

1. **Registry — the single source of truth for what the app *has*.** `src-tauri/src/registry/`. Every capability, setting, permission requirement, nav item, hotkey and metric is one entry. Adding a feature is an entry, not a new pattern. If you are writing a `match` on a feature name anywhere outside `registry/`, the branch belongs in the registry instead.
2. **Command factory — the heart of the app.** `src-tauri/src/ipc/factory.rs`. Every IPC command goes through it. It already does a ton of heavy lifting: input validation, permission preflight, reentrancy guarding, tracing, error mapping and metrics are all handled there, so don't re-check any of it in a handler. Never write `#[tauri::command]` directly.
3. **Commands and pipeline** consume the factory and hold business logic — validation decisions, orchestration, sequencing. Because the factory already covered the cross-cutting concerns, a handler should only contain logic specific to the task at hand.
4. **Ports** are traits for anything swappable. Traits and capability structs only — no logic, ever.
5. **Adapters** are the third-party integrations behind those ports. Only the selected adapter is ever constructed. Never branch on an adapter's name; branch on the capabilities it declares.
6. **Services** are the only layer that touches the database. One verb, one table, no business rules, no calling another service. Import them into commands with `use crate::services::x` so you get module references.
7. Remember to wire up any permission and any metric the feature requires — both are registry entries.

**Never create `middleware.ts`,** and never add a second place where recording state lives. The session state machine owns it.

## 4. Production-grade Rust and TypeScript

Never use `any`, `as any`, `@ts-ignore`, `unwrap()`, `expect()`, or `panic!` outside tests, or any other type or error bypass — this is a production application. Clippy runs at deny level and TypeScript is strict.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webprodigies/murmur](https://github.com/webprodigies/murmur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
