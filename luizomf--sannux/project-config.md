---
trigger: always_on
description: AI context for this repository. Read this before changing files.
---

# sannux Agent Guide

AI context for this repository. Read this before changing files.

---

## Communication / Language

- **Tone:** direct, practical, no corporate fluff.
- **Chat language:** match the user's language.
- **Project language:** English for code, git, GitHub, docs, issues, PRs, and
  `README.md` (except `README-PT-BR.md` files, always keep Portuguese docs up to
  date).

---

## Project Context

`sannux` means **Sandbox Linux**.

This repository holds Docker templates for running AI coding agents in isolated
containers. Each template should be self-contained enough to copy out of the
repo and run on its own.

Users may copy one of these templates to a VPS and run an autonomous or
semi-autonomous agent there.
Assume many users will keep the default shape of the template and only edit the
minimum required variables.

Main goals:

- Keep agent credentials and state isolated from the host machine.
- Mount only the intended workspace into the container.
- Run agents as non-root users by default.
- Provide practical templates, not a production platform.
- Support separate agents with separate workspaces, agent homes, tools, and
  provider keys.

This is a security-sensitive sandbox/tooling repo, but it is also experimental.
Prefer simple, inspectable Docker and shell code over clever abstractions.

---

## Workflow

Use the workflow that fits the change.

- For preserved project work: issue -> branch -> small conventional commits ->
  PR -> merge.
- For small docs tweaks, local experiments, or throwaway sandbox work: a branch
  and PR are optional. Keep the handoff clear.
- Before larger feature work, check open issues first. Reuse an existing issue
  when it matches the request; otherwise create or propose one.
- One focused task per session is the default.

Commit style:

```text
type(scope): short imperative description

Optional body explaining what changed and why.
```

Use conventional commit types such as `feat`, `fix`, `docs`, `chore`,
`refactor`, `test`, and `ci`.

---

## Agent Operating Rules

These are hard rules:

- Read the relevant files before changing anything.
- Make the smallest safe change that solves the request.
- Do not silently rewrite architecture, rename public paths, or move files.
- Do not add dependencies, services, frameworks, build tools, queues, caches, or
  ORMs without a clear reason.
- Do not delete comments, tests, docs, or examples unless they are wrong or the
  user asked for it.
- Do not claim something works without running the relevant check, or clearly
  saying why it was not run.
- Do not commit `.env`, secrets, tokens, private keys, credentials, local agent
  state, logs, caches, or generated throwaway output.
- No destructive git operations without explicit user confirmation.

If a request conflicts with these rules, stop and explain the tradeoff plainly.

---

## Engineering Defaults

Treat these as strong preferences, not bureaucracy.

- Prefer boring, explicit, maintainable code.
- Follow the existing conventions of the language, framework, and local files.
- Prefer editing existing code over creating parallel implementations.
- Prefer clear domain names over vague names like `data`, `handler`, `manager`,
  `processor`, `service`, `utils`, `helper`, `thing`, or `item`.
- Avoid duplication of business rules, validation rules, and non-trivial
  transformations.
- Prefer early returns and shallow nesting.
- Keep functions and files focused, but do not split simple linear code just to
  satisfy a line count.
- Add abstractions only when they remove real complexity or match an existing
  project pattern.
- Preserve public behavior unless the task explicitly asks for a change.

For this repo specifically:

- Template folders under `templates/<template>/` should remain self-contained.
- Root-level helpers such as `justfile` are convenience only; templates should
  still work with plain `docker compose`.
- `docs/template-contract.md` is the canonical template contract. Read it before
  changing templates, and run `just check` before finishing template or contract
  documentation changes.
- Templates must require explicit `WORKSPACE_PATH` and `AGENT_HOME_PATH` values.
  Do not default these paths to folders inside this repository.
- Keep the happy path safe for users who barely read the docs. The template
  should fail fast when required host paths are missing instead of silently
  creating repo-local workspaces or agent homes.
- Dockerfiles and compose files should stay readable and easy to audit.
- Security hardening should be practical and documented. Avoid "paranoid" setup
  that makes the template hard to run unless the task asks for it.
- When adding generated files, caches, agent homes, or logs, check `.gitignore`.

---

## Types, Inputs, and Errors

- Use explicit types at public boundaries when the language supports it.
- Avoid `any` or loose dictionaries when the expected shape is known.
- Validate external input at the boundary: CLI args, env vars, paths, URLs,
  compose variables, API responses, and user-controlled data.
- Error messages should include useful context: what failed, the offending value
  when safe, and what was expected.
- Preserve original errors when wrapping exceptions.

---

## Tests and Verification


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luizomf/sannux](https://github.com/luizomf/sannux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
