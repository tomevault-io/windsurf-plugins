---
trigger: always_on
description: Use the prim CLI for managing Primitive specs, contexts, projects, and pre-commit hooks. TRIGGER when the user mentions Primitive, prim, "specs" (in the Primitive sense), or "contexts" (in the Primitive sense); when the repo's package.json depends on @primitive.ai/prim; when the user asks to sync, map, update, or auto-map a spec; when configuring Primitive pre-commit hooks. SKIP when "spec" means test specs (vitest, jest, rspec), when "context" means React context or LLM context window, or for u
---


# Working with the prim CLI

`prim` is the official CLI for [Primitive](https://app.getprimitive.ai). Use it -- don't reach for shell or curl.

## Mental model

A **spec** captures intent for execution -- it defines what should be done, usually so other agents (or humans) can act on it. A **context** is everything else: supporting material that informs but doesn't define the work -- design docs, references, prior art, shared documentation, examples. When deciding which to create, ask: does this say *what to do*, or does it *inform* whoever's doing it? A project has at most one spec but can link many contexts.

In Primitive, a markdown spec is associated with a **project**. The spec is the source of truth: `npx --yes @primitive.ai/prim spec sync` parses the spec, diffs it against the project, and **applies the diff** -- adding, updating, or **archiving** items in the project to match. Items removed from a spec are soft-archived (recoverable via the dashboard), not deleted -- but they leave the active view, so flag the user before large spec rewrites on projects with work in flight.

A **spec is a kind of context** -- same IDs, same storage. The `npx --yes @primitive.ai/prim spec ...` commands are a focused view onto specs; `npx --yes @primitive.ai/prim context get <id>` works on a spec ID and vice versa. For structured metadata on a spec (review status, root project, sync version, scope, file patterns), use `npx --yes @primitive.ai/prim context get <specId>` -- it returns JSON.

`npx --yes @primitive.ai/prim spec list` returns only spec-type contexts. `npx --yes @primitive.ai/prim context list` returns all contexts regardless of type.

## Auth

Run `npx --yes @primitive.ai/prim auth status` first. It exits **0 if authenticated, 1 if not** -- branch on the exit code, don't parse the message.

Three ways to authenticate, in priority order:

1. **`PRIM_TOKEN` environment variable** -- preferred for agents and CI. Set it before invoking prim and you're done; no interactive flow, no token files.
2. **`npx --yes @primitive.ai/prim auth set-token <token>`** -- saves a bearer token to `~/.config/prim/token`. Use when the user has a long-lived token in hand.
3. **`npx --yes @primitive.ai/prim auth login`** -- opens a browser via WorkOS OAuth. **An agent cannot complete this.** If `auth status` exits non-zero and `PRIM_TOKEN` is unset, **stop and ask the user** to run `npx --yes @primitive.ai/prim auth login` themselves.

The CLI auto-refreshes expired tokens. On unrecoverable expiry it throws `Authentication expired. Run prim auth login to re-authenticate.` -- relay it.

## Ground rules

1. Don't guess IDs. Discover them with `npx --yes @primitive.ai/prim spec list`, `npx --yes @primitive.ai/prim spec list --project-id <pid>`, or `npx --yes @primitive.ai/prim context list`.
2. Every command accepts `--help`. When unsure of flags, run `npx --yes @primitive.ai/prim <cmd> --help` rather than guessing.
3. The CLI prints API errors as one-liners to stderr and exits non-zero. Treat any non-zero exit as actionable. If a command fails with an unrecognized error, re-run with `--help` to check your flags. If auth-related, re-check `auth status`.

## Common workflows

### Read a spec's current text (do this before any partial edit)
```
npx --yes @primitive.ai/prim spec get <id> --text-only > spec.md
```
`npx --yes @primitive.ai/prim spec update <id> --file <path>` replaces the entire body. Fetch first if you're only changing part of it.

### Update a spec from a local file and apply to the project
```
npx --yes @primitive.ai/prim spec list --project-id <pid>     # find the spec for a project
npx --yes @primitive.ai/prim spec update <id> --file spec.md  # replaces spec body
npx --yes @primitive.ai/prim spec sync <id>                   # required -- update doesn't apply changes to the project
```
`npx --yes @primitive.ai/prim spec sync` is **async**: it returns immediately with `Triggered sync for spec`, then applies in the background. The project isn't updated when the command returns -- surface that to the user.

Auto-map runs automatically on the server after every `spec update`. Call `npx --yes @primitive.ai/prim spec auto-map <id>` explicitly only to re-run mapping without changing the spec text.

### Map files to a spec (so pre-commit auto-syncs all affected specs)
```
npx --yes @primitive.ai/prim spec map <id> -p "src/auth/**" "src/foo/**"   # multiple patterns at once
npx --yes @primitive.ai/prim spec unmap <id> -p "src/auth/**"              # remove one
npx --yes @primitive.ai/prim spec unmap <id>                               # clear all manual patterns
```

### Create or link a context
```
npx --yes @primitive.ai/prim context create -s project -n "<name>" --file <path> --project-id <pid>   # add --spec to make it a spec

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [campus-ai/prim](https://github.com/campus-ai/prim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
