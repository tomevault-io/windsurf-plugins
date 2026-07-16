---
trigger: always_on
description: You are running with the tailtest plugin. Your job: automatically run the test cycle the user would otherwise ask for manually. Generate production-like scenarios for what was just built, execute them, and surface only what fails.
---

# tailtest

You are running with the tailtest plugin. Your job: automatically run the test cycle the user would otherwise ask for manually. Generate production-like scenarios for what was just built, execute them, and surface only what fails.

---

## Step 0: Verify APIs before writing test code

Before writing any test code, read the source file and confirm:
- Every class, function, and method you plan to call in the test actually exists in the module
- Every import resolves to a real name in the file

Scope: verifies imports resolve and named attributes exist. Does not validate full call signatures.
If a method does not exist, do not invent it -- adjust the scenario to test what actually exists.

---

## Step 1: At the start of every user turn, check for pending work

Read `.tailtest/session.json`. If `pending_files` is non-empty:

**Before generating:** re-read the source file to understand what it actually does. Derive scenarios from the source's intent and behaviour -- not from your implementation plan or assumptions about what should exist.

1. Note the pending files list
2. Skip any filtered files (filter rules in Step 2)
3. If nothing remains after filtering: clear `pending_files` to `[]`, proceed to the user's message
4. **Before generating, check for ramp-up framing:** if `session.json` has `"ramp_up": true`, count the entries whose `status` is `"ramp-up"` (call this N). If N > 0 **and all remaining pending_files entries have `status: "ramp-up"`** (pure ramp-up batch, no mixed new-file entries), emit exactly one line now: `tailtest: running initial coverage scan on {N} file(s)...` If the batch is mixed (contains both `"ramp-up"` and `"new-file"` entries), skip the framing line.
5. Generate scenarios for all remaining files as one unit of work -- treat `"ramp-up"` entries identically to `"new-file"` (generate scenarios, write test, execute, report)
6. Write test file and execute (Steps 4–5)
7. Report failures -- stay silent if all pass (Step 6)
8. Write `"pending_files": []` back to `.tailtest/session.json`
9. Then address the user's message

**Note:** if a framing line was emitted in step 4, still emit the Step 6 result line normally after execution. Both lines appear in the same turn:
```
tailtest: running initial coverage scan on 7 file(s)...
tailtest: 12 scenarios -- all passed.
```

Treat all pending entries as one cohesive unit. If Claude wrote a service, a model, and a controller this turn, generate one scenario set for what the system does -- not three disconnected sets.

Write **ONE test file** for the entire batch, named after the primary source file (the entry point, controller, or most feature-rich file). Do not create separate test files for models, helpers, or type-only modules -- cover them through the primary file's tests. After writing the test, record every source file in the batch in `session.json`:
```json
{ "generated_tests": { "src/service.py": "tests/test_service.py", "src/model.py": "tests/test_service.py" } }
```

---

## Step 2: Filter -- when to skip

Skip a file with no output if it matches any of these:

**By extension:**
- Config: `*.yaml`, `*.yml`, `*.json`, `*.toml`, `*.env`, `*.ini`, `*.lock`
- Docs: `*.md`, `*.rst`, `*.txt`
- Templates: `*.html`, `*.jinja`, `*.jinja2`, `*.ejs`, `*.hbs`
- GraphQL schemas: `*.graphql`, `*.gql`
- Infrastructure: `*.tf`, `*.hcl`, `*.tfvars`, `Dockerfile`, `*.dockerfile`
- Build tool configs: `*.config.js`, `*.config.ts`, `*.config.mjs`

**By path:**
- `node_modules/`, `.venv/`, `dist/`, `build/`, `generated/`, `.git/`, `vendor/`
- `migrations/`, `db/migrate/`, `database/migrations/`

**By file name:**
- Contains `test_`, `.test.`, `.spec.`, `_spec.`, `_test.`, `Test.`, `Tests.` -- it is a test file

**By file content:**
- TypeScript/JS: contains only `interface`, `type`, `enum` declarations and no function or class bodies
- TypeScript/JS: contains only `export * from` or `export { X } from` statements (re-export barrel)
- Diff is under 5 lines and introduces no new functions or classes

**By project context:**
- Framework boilerplate: `manage.py`, `wsgi.py`, `asgi.py`, `__main__.py` in web projects
- Browser extension project (root contains `manifest.json` with `manifest_version`): skip all files
- Next.js Server Component: file has no `'use client'` directive at top + `next` is in `package.json`
- Next.js Server Action: file has `'use server'` directive at top + `next` is in `package.json`
- Next.js edge runtime: `middleware.ts` or `middleware.js` at project root + Next.js detected

If in doubt whether a file has testable logic, skip. A missed file is better than noise.

---

## Step 3: Generate scenarios

**Output the SCENARIO PLAN first.** Before writing any test code, output the full scenario list in plain English labeled exactly as:

```
SCENARIO PLAN (not final test code):
1. ...
2. ...
```

Do not write test code until the scenario list is complete. Each scenario may generate one or more test cases.

---

Write scenarios that describe business behavior, not function signatures.

**Write this:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [avansaber/tailtest](https://github.com/avansaber/tailtest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
