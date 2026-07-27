---
trigger: always_on
description: 4-implementation: Dev Story
---


# Dev Story

# Dev Story

**Goal.** Implement one story under TDD discipline. Tests first; minimum code to pass; refactor with green. Each commit cites an AC ID. The PR ends in a clean gate, not a "looks good to me."

Shuri drives. Hawkeye observes (test design is binding). Tony stays available for architectural questions.

## Operating contract

I work inside a repo with WDK installed: `.wize/`, `AGENTS.md`, and the `wize-*` skills are my instructions and memory, not background reading. This section is the execution slice of the story's **mission contract** (see `/wize-help mission`).

- **Inspect before editing.** Read the story, its sources of truth, and the touched code first.
- **Reuse ladder before new code** (see `wize-agent-dev` persona): needs to exist → already here → stdlib → framework feature → installed dep → one-liner → only then new code.
- **Test-first, unconditionally.** Red before green, per the loop below — this is the strict-TDD workflow; the "when applicable" latitude lives in `wize-quick-dev`, not here. Smallest sufficient change; follow existing conventions.
- **Run real commands.** No success claim without evidence — a passing run, not "should pass."
- **Never stop at planning.** Planning is a step, not a deliverable.

## Inputs

- `.wize/solutioning/stories/{epic}/{story}.md`
- `.wize/solutioning/architecture.md`
- `.wize/solutioning/design-system/` (use existing components)
- `.wize/implementation/tea/{epic}/{story}/design.md` (the test contract)

## Outputs

- Code + tests in the target repo.
- Commit messages reference AC IDs.
- Story file updated to `status: ready-for-review`.

## The loop (TDD, story-scoped)

### 1. Read

Story ACs, out-of-scope, notes, `tea-design.md`. Confirm the test split. If the design doesn't match the story (mismatch on test count, mock, environment), ping Hawkeye **before** writing code.

### 2. Slice the story into micro-cycles

Each cycle is one AC (or part of one) and produces a green test + small code change. Don't try to ship the whole story in one commit.

### 3. Red

Write the failing test first. Match the assertion shape in `design.md`. Don't write code yet.

```ts
// red — first run, fails
test('valid email returns ok', () => {
  expect(validateInviteEmail('a@b.co')).toEqual({ ok: true });
});
```

### 4. Green

Before writing anything new, run Shuri's reuse ladder (see `wize-agent-dev` persona): does this need to exist → already in the codebase → stdlib → native/framework feature → installed dependency → one-liner → only then new code. Write the **minimum** code that turns the test green. No anticipated branches; no "just-in-case" handling.

```ts
// green — minimum to pass
export const validateInviteEmail = (s: string) => ({ ok: /@/.test(s) });
```

### 5. Refactor

Now harden, with the test as safety net.

```ts
// refactor — same green tests, real logic
import { z } from 'zod';
const schema = z.string().email();
export const validateInviteEmail = (s: string) =>
  schema.safeParse(s).success ? { ok: true } : { ok: false, code: 'invalid_format', field: 'email' };
```

### 6. Commit

Conventional commits, AC IDs referenced.

```
feat(invite): validate email per AC-02-1 / AC-02-2
- add validateInviteEmail with zod
- add 4 unit tests covering valid + invalid rules
```

### 7. Repeat

Until every AC has at least one test + minimum code that makes it pass.

### 7.5. Loop verification (auto-check)

Before declaring the loop done, run these checks:

| Check | How | Fail → |
|---|---|---|
| **Evidence of iteration** | `git log --oneline` shows ≥2 commits with AC IDs (skip if story has only 1 AC) | Return to step 2 |
| **AC-to-test mapping** | Every AC in the story file has a corresponding test case name | Return to step 3 (Red) |
| **All AC tests green** | `npx vitest run --reporter=verbose` (or equivalent) — grep output for each AC's test name | Return to step 4 (Green) |

**Max-cycles guard.** If the same AC cycles through Red→Green→Refactor 3+ times without staying green, stop and escalate to Wizer: the test design may be wrong, the AC may be mis-sized, or the approach may need Tony.

### 8. Knowledge update (inline, ~60s)

Before opening the PR, ask: **did this story touch any of the 5 baseline axes** documented in `.wize/knowledge/document-project/`?

| Axis | Touched when… | File to update |
|---|---|---|
| **Architecture** | new component, new sequence, changed data flow | `architecture-snapshot.md` |
| **Conventions** | new naming/folder/test pattern published as public contract (incl. `testid`) | `conventions.md` |
| **Risk-spots** | introduced a complexity hot spot OR resolved one | `risk-spots.md` |
| **Dependencies** | added / removed / upgraded a runtime dep | `dependencies.md` |
| **Overview** | new user-visible feature a new dev should know about | `overview.md` |

If yes for any axis: open the file and add **1–3 lines** under a new dated bullet — in the same PR.

```markdown
## 2026-06-12 — E01-S03
- Conventions: `data-testid="invite-*"` published as public contract; Hawkeye E2E depends on these.
- Risk: R-1 (mailer) mitigation now confirmed (integration test covers retry policy).
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qwize-br/wize-development-kit](https://github.com/qwize-br/wize-development-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
