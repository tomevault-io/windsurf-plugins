---
trigger: always_on
description: Primary AI instruction file for this repo — and the human contributors' source of truth.
---

# ObjectStack — AGENTS.md

Primary AI instruction file for this repo — and the human contributors' source of truth.
Read natively by Claude Code, GitHub Copilot (coding agent + CLI), and other agents — no
separate `.github/copilot-instructions.md` mirror needed. When any other instruction file
in this repo (including `.claude/skills/**`) conflicts with this one, **AGENTS.md wins**.

> **v5.0 breaking rename: `project` → `environment`** everywhere (CLI `-e`, `/api/v1/environments/:id`, header `X-Environment-Id`, `OS_ENVIRONMENT_ID`, DB column `environment_id`). No aliases. See ADR-0006. "Project" now only means the npm/monorepo sense.

This file carries principles, binding rules and lookup tables. Lessons from past
incidents are distilled in place — failure mode, discipline, boundary — without
issue-number citations (maintainer ruling, 2026-08-12: 「处理 issue 时犯的错应该总结成
经验,保留 issue id没有意义」); maintainer rulings keep their date and verbatim quote.
Where a hook or CI gate enforces a rule mechanically, the rule is stated once here and
the script's own header is the authority on detail.

---

## Communication

语言规则分两件事:**和维护者说话用什么语言**,与**留在 GitHub 上的产物用什么语言**。
它们可分,并且就在这里分开。

- **在 Claude Code 中与维护者对话一律使用中文**(对话回复、轮次报告等聊天通道里的内容)。
- **GitHub 产物一律使用英文**:issue 与 PR 的标题、正文、评论。维护者裁决
  (2026-08-08),原文引用、未翻译:

  > issue 和 PR 必须用英文，在 claude code 中和我讨论可以用中文。

- **引用中文裁决时保持原文、不翻译**,即使承载它的 issue/PR 正文通篇是英文——改写引文
  就是改写裁决。上面那段引用即是一例:它是维护者的原话,一个字未动。
- 代码、标识符、提交信息(commit messages)、ADR/文档正文等仓库产物保持现有语言惯例(以英文为主),不要因本节而改写。

The rules are split per channel because a merged rule was measured to fail: an agent
holding one instruction that claimed explanatory PR text for Chinese and another
demanding English produced half-Chinese, half-English PR bodies on the same day. One
rule per channel, no overlap.

---

## Build & Test

```bash
pnpm install          # deps
pnpm setup            # first-time: install + build spec
pnpm build            # turbo build (excludes docs)
pnpm test             # turbo test
pnpm typecheck        # turbo typecheck — per-package `tsc --noEmit`; tsup/vitest never type-check
pnpm docs:dev         # docs site
```

Type-check coverage and its debt counts are ratcheted in CI
(`pnpm check:type-check-coverage`, `pnpm check:type-check-debt`; the script headers are
the authority on detail): every package declares a `typecheck` script or carries a
measured, shrink-only DEBT/EXEMPT ledger entry; new packages arrive covered; a package
that graduates deletes its entry in the same PR; when a re-measure forces a count up,
rewrite the entry's `note` too — a note naming only the old errors reads as "nearly
graduated" to the next author.

Three principles the ratchet's invariants encode, worth knowing before you fight them:

- **Never `exclude` `*.test.ts` / `*.spec.ts` from a package's `tsconfig.json`** —
  `tsc --noEmit` reads that config, so the exclusion hides the tests from the very
  check the `typecheck` script advertises (a green gate over source nothing read). When
  the build config must keep the exclusion, add a sibling `tsconfig.test.json` and name
  it in the `typecheck` script (the `packages/spec` pattern); the sibling may carry its
  own *module* semantics to match vitest, never its own *strictness*.
- **A `@ts-expect-error` in a file no tsc program compiles is a phantom check** — it
  evaluates never, and deleting it leaves every gate just as green (a repo-wide sweep
  once found seventeen retirement pins in that state in one package alone). Before
  writing one, check the file is compiled. Test-layer residue lives in the per-file,
  shrink-only `<package>/test-typecheck-debt.json` (regenerate with
  `pnpm --filter <package> gen:test-typecheck-debt`); the shared gate is
  `scripts/check-test-typecheck.mts --package <dir>` — onboard by wiring, never by
  copying.
- **A pile of TS7006 "implicitly any" is usually one broken import upstream**, not a
  package that needs annotations: under `moduleResolution: NodeNext` a relative import
  missing its `.js` extension does not resolve and every symbol it names becomes
  `any`. Fix the extension first and re-measure.

### Running the dev server

| Scenario | Command | Notes |
|:---|:---|:---|
| **Frontend debug** (UI in `../objectui` calls backend) | `PORT=3000 pnpm dev` | `pnpm dev` = the **showcase** kitchen-sink app (default; best for exercising the platform). Port **must** be 3000 (UI hard-wired); persistent state; leave running. For the minimal CRM app instead: `PORT=3000 pnpm dev:crm`. |
| **Backend-only debug** | `pnpm dev -- --fresh -p <random>` | Random high port; ephemeral tempdir; **you must kill it** when done |

`--fresh`: ephemeral tempdir (auto-deleted on exit) + `--seed-admin` (POSTs sign-up, prints creds — default `admin@objectos.ai` / `admin123`, override via `--admin-email`/`--admin-password`). The seeded admin is auto-promoted to **platform admin** (the system seed identity `usr_system` is skipped), so Setup/Studio are reachable on first login.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [objectstack-ai/objectstack](https://github.com/objectstack-ai/objectstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
