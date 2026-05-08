---
trigger: always_on
description: OpenErrata is a browser extension that investigates the content people read on the
---

# OpenErrata

OpenErrata is a browser extension that investigates the content people read on the
internet with LLMs, and provides inline rebuttals to empirically incorrect or
unambiguously misleading information. The design goals, architecture, data model,
and API are documented in `SPEC.md` — that is the source of truth for product
behavior.

## Repository Layout

```
src/
├── helm/openerrata/         # Helm chart (single deployment artifact for on-prem + hosted)
└── typescript/
    ├── shared/              # @openerrata/shared — types, Zod schemas, normalization
    ├── api/                 # @openerrata/api — SvelteKit + tRPC backend, Prisma, job queue
    ├── extension/           # @openerrata/extension — Chrome MV3 browser extension
    └── pulumi/              # @openerrata/pulumi — deploys the Helm chart for hosted env
```

The monorepo uses pnpm workspaces. Dependencies flow: `shared` → `api` and
`shared` → `extension`. The extension imports the API's `AppRouter` type (type-only,
no runtime code) for tRPC client typing.

## Key Files

- `SPEC.md` — Product spec. All product behavior questions are answered here.
- `src/typescript/api/prisma/schema.prisma` — Database schema (source of truth
  for data model).
- `src/typescript/api/src/lib/investigators/prompt.ts` — The LLM investigation
  prompt.

## Workspace Guidelines

Assume that other AI agents are working inside the same directory as you. When
instructed to make commits, take care not to accidentally or unintentionally
commit the work of others. Assume questions and requests are about your changes
by default.

## Programming Guidelines

Code can be "correct" in the sense of doing what we want, and yet still be bad
in the sense of encouraging further mistakes, making it harder for other AI
agents to read or iterate on your code, or making it unclear what the intent
behind it was. Here are some rules so you do not write code that is bad in
these ways.

Always take the time to introduce new features and fix bugs in the codebase in
the most complete, ideal, and maintainable way, even if it means modifying more
than your user initially expects. Feel free to perform refactors of bad code
not originally mentioned in your prompt. 

**The underlying driving spec behind the software you're tasked with writing
should be clearly understandable just by reading your code.** This is the most
important rule. As the product evolves, a larger and larger proportion of the
spec will live outside of the SPEC.md and implicitly in the code. If it is not
immediately clear what the code you are writing is designed to do, refactor it
until that is apparent just from its structure. Anyone looking at your code
should be able to tell why it is designed that way and what purpose it serves.
This helps AI models and humans that have to reread the codebase and infer this
information from scratch.

Treat the public interface of code you're writing or using as advertising an
implicit or explicit "spec", distinct from its internal implementation. As
previously stated, the spec should be clear, consumers should depend only on
the spec and not incidental implementation details, and consumers should
support the whole range of behavior that the spec suggests rather than just
what the code exhibits at the present date.

When modifying code you didn't write, do not maintain legacy behavior that you
can't identify a design motivation for. Clearly explain any differences in
behavior that your operator might not be aware of, but otherwise feel free to
make sensible design changes if they serve the purpose of making the code more
maintainable.

When designing data structures and functions, adhere to the representable-valid
principle as much as possible. Do not tolerate datatypes, structs, or database
tables with possible "invalid" configurations. Don't even let groups of
variables accessible from the same namespace have invalid configurations. Each
"invalid" datatype value is a possible failure state for the program to enter
into, and another requirement of the application for data accessors to reason
about, that could be offloaded to the type checker.

Bad:

```
type StreetLightStatus = {
  red_light: bool
  yellow_light: bool
  green_light: bool
}
var asyncLoadedData: Data | undefined = undefined
var asyncLoadedDataError: str | undefined = undefined
```

Better:
```
type StreetLightStatus = enum {
  RED
  YELLOW
  GREEN
}
var asyncDataState: {
   state: "loading"
} | {
   state: "errored"
} | {
   state: "loaded"
   data: Data
}
```

For untyped languages that do not support strict type definitions but still
support 'type hints', you should use those to express constraints.

Variables should always contain an accurate copy of the data they advertise in
their name at each program step. Never populate datatypes with "dummy" or
placeholder values, even temporarily, because that creates a condition for
access that other programmers have to reason about. Neither should you return
objects or values that would give callers a literally incorrect state of the
world. If you catch yourself doing this, come up with a refactor that does not
require you to do so instead.

When key assumptions that your code relies upon to work appear to be broken,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZeroPathAI/OpenErrata](https://github.com/ZeroPathAI/OpenErrata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
