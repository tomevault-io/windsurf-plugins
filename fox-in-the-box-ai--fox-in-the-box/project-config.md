---
trigger: always_on
description: This file governs how Cursor (and any AI coding agent) works in this repository.
---

# Fox in the Box — Cursor Development Instructions

This file governs how Cursor (and any AI coding agent) works in this repository.
Read it in full before writing any code.

---

## 1. Your Role

You are the **implementer**. You write code, tests, and config files.
You do NOT commit, push, or open PRs on your own.

A separate **Supervisor agent (Hermes)** reviews all changes before they are committed.
The workflow is: **you implement → Supervisor reviews → Supervisor commits**.

Do not bypass this. Do not `git push` under any circumstances.

**You MUST make a WIP draft commit** when you are done (see Section 6 — Signal Done).
This protects your work from being lost if the worktree directory is touched.
The Supervisor will amend the commit message and push.

---

## 2. Repository Layout

```
fox-in-the-box/             ← monorepo root
├── forks/
│   ├── hermes-agent/       ← git submodule (fox-in-the-box-ai/hermes-agent)
│   └── hermes-webui/       ← git submodule (fox-in-the-box-ai/hermes-webui)
├── packages/
│   ├── integration/        ← Dockerfile, supervisord, entrypoint, default-configs
│   ├── electron/           ← Electron desktop app
│   └── scripts/            ← install.sh, dev utilities
├── docs/
│   ├── tasks/              ← One task doc per feature (your specs live here)
│   ├── archive/            ← Frozen design docs (REQUIREMENTS.md, ROADMAP.md)
│   └── GATEWAY.md, RESET.md, GETTING_STARTED.md, …
├── tests/
│   ├── integration/        ← Full-stack tests
│   ├── electron/           ← Electron unit tests (jest)
│   └── container/          ← Shell/bats tests
├── README.md               ← Live roadmap lives in README's Roadmap section
└── AGENTS.md               ← This file
```

Submodules are in `forks/`. Do not modify submodule content directly.
If you need to change `hermes-agent` or `hermes-webui`, say so explicitly in your output —
the Supervisor will handle the fork workflow.

**Docker image:** `packages/integration/Dockerfile` **COPY**s `forks/hermes-agent` and `forks/hermes-webui` into the image and runs `pip install` at **build** time. That is normal integration work (not “editing forks” in place). A local `docker build` **fails** if submodules are not checked out — run `git submodule update --init --recursive` first (CI already uses recursive submodules). Runtime entrypoint **does not** clone those repos from GitHub for production containers.

**Line endings:** `.gitattributes` keeps `packages/integration/**/*.sh` as **LF** so Windows checkouts do not break Linux shebangs inside Docker.

---

## 3. Git Worktrees

Each task runs in its own **git worktree** to keep work isolated.

```bash
# Supervisor creates the worktree for you before handing you the task:
git worktree add ../fitb-task-03 -b task/03-dockerfile

# You work inside that directory:
cd ../fitb-task-03

# When done, you signal completion. Supervisor reviews and commits.
# Supervisor merges back to main and removes the worktree.
```

Rules:
- One task = one worktree = one branch (`task/NN-short-name`)
- Never work on `main` directly
- Never switch branches inside a worktree
- Leave the worktree in a clean, buildable state when done

---

## 4. Task Workflow

For every task you receive:

1. **Read the task doc** (`docs/tasks/NN-taskname.md`) in full before writing any code
2. **Read `docs/archive/REQUIREMENTS.md`** — frozen design doc that captures the broader-system intent. Cross-check anything material against the current code; the design doc is from before v0.1 and many specifics have evolved.
3. **Write tests first** (TDD) — every acceptance criterion in the task doc maps to at least one test
4. **Make tests pass** — implement until all AC tests are green
5. **Self-review checklist** (before signalling done):
   - [ ] All acceptance criteria tests pass
   - [ ] No secrets, API keys, or credentials in any file
   - [ ] No hardcoded paths that differ between dev and container (`/data` not `~/whatever`)
   - [ ] Code follows existing patterns in the repo (check neighboring files first)
   - [ ] No `console.log` / `print` debug statements left in
   - [ ] `docker build` still succeeds if you touched anything in `packages/integration/`
6. **Signal done** — do these steps in order:
   a. Write `DONE.md` in the worktree root with:
      - What you implemented
      - How to run the tests
      - Any open issues or assumptions you made
      - Anything that needs Supervisor's attention
   b. **Make a WIP draft commit** (this is mandatory — not optional):
      ```bash
      git add -A
      git commit -m "WIP: task(NN) — pending supervisor review"
      ```
      This protects your work. The Supervisor will amend the message on approval.
      Do NOT `git push`. The Supervisor handles all pushes.

---

## 5. Testing Standards

### Python (backend, entrypoint logic)
```bash
pytest tests/ -v
```
- Use `pytest` + `unittest.mock` for mocking subprocesses and filesystem
- All `/api/setup/*` endpoints must have unit tests (see task 05)
- Test files: `tests/integration/test_*.py`

### JavaScript (Electron)
```bash
cd packages/electron && pnpm test
```
- Use `jest` for unit tests
- Mock `dockerode` — never require a live Docker daemon for unit tests
- Test files: `tests/electron/*.test.js`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fox-in-the-box-ai/fox-in-the-box](https://github.com/fox-in-the-box-ai/fox-in-the-box) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
