---
trigger: always_on
description: Shared instructions for all AI agents (Claude, Codex, etc.).
---

# [AGENTS.md](http://AGENTS.md)

Shared instructions for all AI agents (Claude, Codex, etc.).

- You are an AI assistant working on the VMark project.

- Use English regardless what languages xiaolai uses.

- Follow the VMark working agreement:

  - Run `git status -sb` at session start.

  - Read relevant files before editing.

  - Keep diffs focused; avoid drive-by refactors.

  - Do not commit unless explicitly requested.

  - Keep code files under \~300 lines (split proactively).

  - Do not destructure Zustand stores in components; use selectors.

  - Prefer `useXStore.getState()` inside callbacks.

  - Keep features local; avoid cross-feature imports unless truly shared.

  - **Research before building**: For new features, search for industry best practices,  
    established conventions, and proven solutions (web search, official docs, prior art in  
    popular open-source projects). Don't invent when a well-tested pattern exists.

  - **Archive finished deep researches**: When a deep-research run finishes, write its full  
    cited report (verdict, findings, refuted claims, caveats, sources) to  
    `dev-docs/deep-researches/YYYYMMDD-topic.md` and link it from `dev-docs/README.md`.  
    Don't leave finished research only in chat transcripts or task output files.

  - **`dev-docs/` and `.vmark/` are maintainer-local (gitignored)**: they exist on  
    maintainer machines, not in the public repo. References to `dev-docs/` in these  
    rules apply when the folder is present; skip them otherwise.

  - **Edge cases are not optional**: Brainstorm as many edge cases as possible — empty input,  
    null/undefined, max values, concurrent access, Unicode/CJK, RTL text, rapid repeated  
    actions, network failures, permission denials. Write tests for every one.

  - **Test-first is mandatory** for new behavior:

    - Write a failing test (RED), implement minimally (GREEN), refactor (REFACTOR).

    - Coverage thresholds are enforced — `pnpm check:all` fails if coverage drops.

    - Exceptions: CSS-only, docs, config. See `.claude/rules/10-tdd.md` for full scope.

  - Run `pnpm check:all` for gates.

  - **`check:all` is the gate, not the loop.** It runs ~1,465 test files, 25
    lint gates and three builds; measured locally it is a ~15-minute round
    trip, which makes it useless as feedback while you work. Use
    `pnpm check:fast` (`typecheck` + cached `lint` + tests related to your
    changes) as the inner loop, and run `pnpm check:all` once before you push.

  - **`pnpm check:predelta` bridges the two — run it before the confirming
    `check:all`.** `check:all` exits on the FIRST failure, so a batch of
    independent `check:all`-only problems surfaces one per 15-minute run — using
    a slow gate for discovery, which this file forbids ("never use a slow gate as
    an instrument of discovery"). `check:predelta` runs **every**
    `check:static` leaf, plus `check:servers` and `check:build` and the
    runtime-file app tests — **in parallel, collecting EVERY failure at
    once** — and skips only the full instrumented app suite (`test:coverage`)
    and `test:changed`, which is a subset of it.

    The property it now guarantees, pinned by `check-predelta.test.mjs`: **a
    green `check:predelta` means `check:all` cannot die in a gate predelta
    could have run.** `lint` used to be excluded on the reasoning that
    `check:fast` covers it — true only if you happen to have run `check:fast`
    since your last edit, which a pre-push gate cannot assume. On 2026-08-21
    predelta reported all 38 gates green and the confirming `check:all` died
    ~40 seconds later on five eslint errors: one full cycle spent discovering
    what a cached, seconds-long gate already knew. eslint is `--cache`d and
    runs in parallel here, so including it costs nothing.

    `typecheck` has no gate of its own and is not skipped either — `check:build`
    → `build` → `typecheck`, so a type error surfaces as a `check:build`
    failure. Measured **1m41s vs ~15min** (2026-08-21, 40 gates;
    it was ~40s before `lint:type-aware` and `lint:test-types` joined
    `check:static` — both are ~95s and run in PARALLEL with each other, so they
    set the floor together rather than adding up). It finds the whole batch in
    one pass (the six issues the "does not see" list below can produce: a
    baseline ratchet, a knip finding, a corpus-enumerating test, a sidecar
    ESM/coverage break, a `size-limit` overflow). The gate list is DERIVED from
    `package.json`, so it cannot drift; `scripts/check-predelta.test.mjs` pins the
    derivation, the collect-all behaviour, and that it stays out of `check:all`
    (it is a pre-push helper, not a CI gate). It does NOT replace `check:all` —
    the full app coverage suite still runs there.

    What to run after a change:

    | What you changed | Run |
    |---|---|
    | One app `.ts`/`.tsx` | `pnpm test:changed`, or `pnpm vitest related <file>` |
    | A store/service with many importers | `pnpm test:changed` — the import graph handles the fan-out |
    | A lint gate under `scripts/` | that gate, plus its own `scripts/*.test.*` file |
    | Locale JSON | `pnpm lint:i18n && pnpm vitest run src/locales` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tyche-MKR/vmark](https://github.com/Tyche-MKR/vmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
