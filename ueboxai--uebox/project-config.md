---
trigger: always_on
description: Instructions for AI coding agents working in this repository.
---

# AGENTS.md

Instructions for AI coding agents working in this repository.
中文版：[AGENTS.zh-CN.md](AGENTS.zh-CN.md)（两份必须同步更新，CI 会检查）

---

## 1. What this project is

**Unreal Box (虚幻盒子)** — an agent harness for Unreal Engine: a local runtime that lets an AI
agent actually operate an Unreal project. Electron + Vue 3 + TypeScript.

The agent is the product. The harness around it is model access, the Unreal tool set, the
skills, and the constraints and verification that keep it in bounds; the engine connection
itself runs through the bundled UnrealAgentLink plugin.

The other modules — project library, asset library, blueprint and material libraries,
notebooks and wikis — each have a full standalone UI, but structurally they exist to serve the
agent: they register what it acts on, feed it material, and receive what it produces. Treat
them that way when deciding where a feature belongs.

This repository is the open-source desktop application. Local features work fully offline,
without an account or region gate. Optional AI and network vault features use services
explicitly configured by the user.

**Never** add telemetry, remote calls, or an account requirement to a local feature.

## 2. Setup

```bash
pnpm install
pnpm dev          # start the app (Electron)
```

Windows is the primary platform. pnpm 10.28.2, Node 24.21.0.
The project pins Node in `.npmrc`; use `pnpm exec node` for direct Node commands so they use
the project runtime even when the system Node differs.

## 3. The gate — two tiers, and which one you owe

**Finishing a single task:**

```bash
pnpm verify:changed     # ~30s — only what you touched
```

**Before the work is published** — you are about to `git push`, cut a release, or hand the whole
batch over:

```bash
pnpm verify             # ~3.5 min — the full gate
```

**Do not run the full gate after every small task.** Those 3.5 minutes are almost all fixed cost:
the full lint ratchet is 49s, both typecheck projects 45s, and the whole test suite 126s — of which
only 43s is actually executing tests (the rest is 386 test files each building an environment).
Paying that per task is the biggest single waste of time in this repo. With several sessions
working at once it is also actively misleading: the gate is repo-wide, so another session's
half-written file turns **your** gate red (§5 rule 11).

`verify:changed` is a snapshot, not a substitute — the full gate still has to pass before anything
is published. Two things make deferring it safe:

- Every task got its own 30-second check, so a red full gate points at _this_ batch, not at a task
  someone finished three hours ago and can no longer reconstruct.
- **CI is the backstop.** `.github/workflows/quality.yml` does nothing but `pnpm verify --ci`, and
  it runs on **every push** — the full gate plus packaging and the offline-boot check. The gate is
  defined in exactly one place (`scripts/verify.mjs`), so local and CI cannot drift. Forgetting to
  run it locally costs you a red run on GitHub, not a broken release.

Every step tells you what to fix when it fails. You can leave the app running while it runs — the
gate no longer touches the `better-sqlite3` native binding.

| Step | What it checks |
|---|---|
| `security:secrets` | No credentials committed |
| `verify:official-endpoints` | Ratchet: the community core must not gain new official-server calls |
| `lint` | Ratchet: full-repo ESLint, per-file counts must not exceed `scripts/lint.baseline.json` |
| `lint:changed` | **Full ESLint rules on the lines you changed** — zero tolerance |
| `docs:check` | Bilingual doc pairs were updated together |
| `verify:skills` | `resources/skills/**` conforms to `resources/skills/SKILL_STANDARD.md` |
| `verify:ue-file-reads` | Engine-authored files (`.uproject` / `.uplugin` / engine `ini`) are never read with a hard-coded encoding — see §5 |
| `plugin:check` | Plugin source changed ⇒ the zip must be rebuilt — this step checks UE 5.5 only |
| `typecheck` | `tsconfig.node.json` (main) + `tsconfig.web.json` (renderer) + `tsconfig.cli.json` (`packages/cli`) |
| `test:run` | Vitest, whole-repo unit tests (no count here — it changes daily and would just go stale) |
| `audit:prod` | CI only — no high-severity vulnerabilities in production deps |
| `build:unpack` | CI only — production build + packaging + packaged-dependency closure |
| `verify:offline-boot` | CI only — boots the packaged app, walks every page, asserts zero official-server requests |

What the per-task tier actually does: `verify:changed` skips the full lint ratchet (new code is
still covered by `lint:changed`, zero tolerance), typechecks only the side you changed, and runs
only the tests related to your diff. Everything else in the table above still runs — those steps
are cheap.

Its comparison base is `$VERIFY_BASE` → `origin/main` → `main` → `HEAD`. So as an unpushed batch
grows, `verify:changed` slowly widens to cover the whole batch. To check strictly the task in
front of you, pin it: `VERIFY_BASE=HEAD pnpm verify:changed`.

Two other variants: `pnpm verify:fast` (the whole gate, but the test step skips the native-ABI
preflight) and `pnpm verify --with-build` (adds packaging and the offline-boot gate locally — use
it when you touched the main process, build config, or dependencies).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ueboxai/uebox](https://github.com/ueboxai/uebox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
