---
trigger: always_on
description: dunx is a Bun-native dependency injection framework, published as ten workspaces
---

# dunx - Claude Code Instructions

dunx is a Bun-native dependency injection framework, published as ten workspaces
under `@dunx/*`.

**Most rules in this repo are enforced by a gate, not by this file.** Write the
change, then run `bun run ci` (35s, every gate CI runs). It names what it wants.
Do not try to satisfy the checks from memory before you have run them once.

The reasoning behind every rule here, including what was measured and what was
tried and reversed, is in
[internal/notes/research/repo-rules-rationale.md](./internal/notes/research/repo-rules-rationale.md).
Read it when you need to know _why_, not to find out _what_.

## The four rules

**Rule 1 - native implementations only.** Every capability is built on a `Bun.*`
API, a Web standard Bun implements, or a native module via N-API. Never
reimplement what Bun already does (`Bun.serve`, `Bun.SQL`, `bun:sqlite`,
`Bun.RedisClient`, `Bun.Glob`, `Bun.password`, `Bun.S3Client`). Never invent what
a mature library already solves: dunx integrates zod, drizzle-orm, better-auth
and bullmq rather than competing with them, and they are **peers**, never
dependencies. A `dependency` of a published workspace may only be `@arkv/*`,
`@dunx/*` or `oxc-parser`.
→ `scripts/rule-native.test.ts`

**Rule 2 - one declaration, at the lowest common owner.** Before writing
anything, search for it. If a second copy would exist, move the first somewhere
both can reach and delete it in the same change. This covers code, types,
constants and styles. Two frontends needing a component means `@dunx/ui`; two
packages needing a traversal means it moves down to the package that owns the
data; a wire format is declared by the server and imported by the frontend with a
relative `import type`.
Not gated. It is the rule most often broken while adding a feature.

**Rule 3 - a package's surface is classes.** In `packages/*` and `tools/*`,
anything with state, configuration or a lifetime is a class, and anything a
consumer injects **must** be a class (the container resolves runtime values; an
interface at an injection site is a boot error). A module is a class with static
`forRoot`/`forRootAsync`. The exception is a pure, stateless, argument-in
value-out function nobody configures. `internal/*` is exempt.
Not gated.

**Rule 4 - a feature is not shipped until an example uses it.** Adding or
changing a capability in `packages/*` or `tools/*` includes updating
`examples/full` in the same change. Do not add to `examples/minimal`.
→ `scripts/rule-example.test.ts`

## Dependency injection

Constructor injection needs no annotation. `@dunx/transform` reads constructor
parameter types at load time and records them as a thunk under
`Symbol.for('dunx.deps')`; apps opt in with `preload = ["@dunx/transform/preload"]`.

```ts
export class UsersService {
  constructor(private readonly repo: UsersRepository) {}
}
```

- A parameter whose type is erased (interface, primitive, union, type-only
  import) is a **boot error naming that parameter**, not a silent `undefined`.
- A class with constructor parameters but no record means the plugin never ran;
  the container detects this via `ctor.length` and fails with the preload
  snippet. Do not make core register the plugin on import.
- The record is a thunk, so a dependency declared later in the file or across a
  circular import works without `forwardRef`.
- `inject()` in a field initializer is the escape hatch and may be mixed in.
- The transform only touches **class declarations**, never class expressions.

**The container is scoped, not flat.** Every module reference is a scope;
`exports` is its public surface and absent `exports` means nothing is exported.
Two consequences that bit real code:

- **A framework service must be bound by a module, not left to self-bind.** An
  unbound class self-binds into whichever scope asks first, so a second consumer
  is a boot error.
- **A module that takes no options is a decorated class, not a `forRoot()`.**
  `forRoot()` returns a fresh object per call, so two importers build two scopes.

Never add code that assumes one flat namespace. Details:
`docs/architecture/dependency-injection.md`.

## Always-bound contracts

`AppFactory.create` binds three tokens **after** every module's, so a module that
binds one wins:

| Token            | Default               | Replaced by                            |
| ---------------- | --------------------- | -------------------------------------- |
| `Logger`         | `ConsoleLogger`       | `LoggerModule` → `@arkv/logger`        |
| `RequestContext` | `AsyncRequestContext` | `LoggerModule` → arkv's `ContextStore` |
| `Tracer`         | `NoopTracer`          | `OtelModule` (`@dunx/core/otel`)       |

No default reaches for a dependency. `ConsoleLogger` batches `info` and
below into one write per event-loop turn; `warn` and above are never batched.

## Configuration

`ConfigModule.forRoot` takes either `{ validate }`, **one validation function**,
or `{ schema }`, a Standard Schema validated directly; there is no schema DSL of
its own. Bun loads `.env` itself, so there is no loader and no `dotenv`.

Declare a subclass and hand it to `as`, which is what keeps the type through a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [petarzarkov/dunx](https://github.com/petarzarkov/dunx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
