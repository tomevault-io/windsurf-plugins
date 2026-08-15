---
trigger: always_on
description: Context for an AI assistant working in this repository. Humans may find it
---

# Notes for coding agents

Context for an AI assistant working in this repository. Humans may find it
useful too, but the audience is a model with no memory of the last session.

Read this file first. Then read the `AGENTS.md` nearest the code you are about
to change: [`server/`](server/AGENTS.md), [`localdrive/`](localdrive/AGENTS.md),
[`docs/`](docs/AGENTS.md), [`landing/`](landing/AGENTS.md). Those hold local
rules only and do not repeat what is here.

For **why** the project makes the choices it does, and for judging whether a
change belongs at all, read [`VISION.md`](VISION.md). This file is how to work.
That one is what to work towards.

## What this is

Local Drive is a self hosted alternative to Google Drive. Two parts:

- `server/` is a Go binary. **This is the product.** It holds the data, serves
  the API, and runs on its own.
- `localdrive/` is a Flutter client for Android, Windows, Linux, macOS, iOS and
  the web. It is an interface to the server and holds nothing the server does
  not.

Also `docs/` (the single source for all documentation) and `landing/` (a
Next.js site that reads `docs/` directly at build time).

Machine readable facts about commands, paths and components are in
[`.ai/project.json`](.ai/project.json). Prefer it over guessing, and update it
when the thing it describes changes.

## The loop

Work in this order. Most bad changes come from starting at step 4.

1. **Discover.** Find the code that already owns this behaviour. Search before
   creating. A second implementation beside an existing one is the most
   expensive mistake available here.
2. **Understand.** Read the surrounding package and its tests. Check `docs/`
   for the documented behaviour, which is the contract.
3. **Plan.** Decide the smallest change that solves it. Note what could break.
4. **Implement.** Match the file you are editing.
5. **Test.** Run the checks below. Add a test when behaviour changed.
6. **Review.** Reread your own diff as a reviewer would. Remove anything that
   is not part of the stated change.
7. **Document.** Update `docs/` when user visible behaviour changed.
8. **Verify.** Run the checks again, on the final state of the tree.
9. **Summarise.** Say what changed, what you ran, what passed, what failed and
   what you could not determine.

## Ground rules

**Do not add dependencies without being asked.** The server builds with
`CGO_ENABLED=0` and a pure Go SQLite driver so it stays one file with no
runtime. A dependency that breaks that trades away the main thing the project
offers.

**Do not restate documentation in a second place.** `docs/` is the source.
README files link to it. If two files describe the same behaviour, one of them
is already wrong.

**Run the checks.** They are fast and they catch real things:

```
cd server     && go test ./... && go vet ./...
cd localdrive && flutter analyze && flutter test
cd landing    && npm run build
```

The landing build parses every documentation file and fails on an internal
link that does not resolve. It is the link checker.

**When the change is user visible, prove it against something running.** The
end-to-end layer runs on TestSprite, and the judgement about when it is worth
the cost is in [`.ai/skills/verify-with-testsprite.md`](.ai/skills/verify-with-testsprite.md).
It cannot reach `localhost`, so a change that is only on this machine cannot be
verified there — say that rather than pointing it at an older deployment. The
checks above still come first, and still catch more per second spent.

**Verify before claiming.** Do not report a command as working without running
it. Several things in this project behave differently under Docker and as the
bare binary, and guessing which is which produces documentation that fails for
the reader.

**Report what you could not do.** A change that says "I could not run the
Flutter tests, no SDK on this machine" is useful. The same change claiming they
passed is not, and it costs the reviewer their trust in the rest of it.

## Never

- Commit a secret, a key, a keystore or a `.env`. Check `.gitignore` before
  adding anything that looks like credentials.
- Bypass an authorisation check, or add an admin path that reads another
  account's files.
- Weaken, skip or delete a test to make a build pass. A failing test is a
  finding, not an obstacle.
- Claim a test passed without running it, or hide a failure in a summary.
- Change the licence, the security policy or the code of conduct.
- Change API responses, CLI output, database schema or on-disk layout as a side
  effect of something else. Those are public contracts. See
  [Changing something public](#changing-something-public).
- Delete user data, or write a migration that cannot be reversed, without being
  asked in those words.
- Refactor code that is not part of the task.

## Always

- Search for the existing implementation first.
- Keep the diff scoped to what was asked.
- Prefer the smallest change that is actually correct.
- Add a test when you change behaviour.
- Update documentation when you change what a user sees.
- Say plainly what you are unsure about.

## Decisions that look like bugs and are not

Change these only with a clear instruction, because each one has a reason that
is not visible from the code alone.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MultiX0/LocalDrive](https://github.com/MultiX0/LocalDrive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
