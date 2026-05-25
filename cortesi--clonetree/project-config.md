---
trigger: always_on
description: - Never commit code without explicit user confirmation. Just because the
---

# Core Guidelines

- Never commit code without explicit user confirmation. Just because the
  user has consented to one commit doesn't mean they consent to all future commits.
- When removing or changing code, never add comments about what was removed or
  changed. Comments in the code should always reflect what's there in the moment.
- You are an autonomous agent. You make use of all the tools available to you.
  You run instrument code and run tests and smoketests to gather information to
  solve problems. You iterate persistently until your requirements are met.
- You may create temporary files and directories as needed to solve problems,
  but always place them in the `./tmp/` directory.
- ALWAYS lint and fix all warnings before returning to the user.
- As a final step, ALWAYS format code before returning to the user.
- Adding an environment variable to configure code, enable/disable features,
  enable debugging output is almost always the wrong approach. Prefer function
  parameters or configuration structs.
- Adding sleeps or timeouts to code is nearly ALWAYS the wrong approach. Prefer
  using synchronization primitives, callbacks, or event-driven mechanisms.
  Never try to "fix" test failures by tinkering with timeouts or sleeps, and
  treat every construction of an arbitrary timeout as a code smell.
- DO NOT over-use conditional compilation. Every time you're tempted to add
  `#[cfg(...)]` to your code, ask yourself if there's a better way to structure
  the code so that it doesn't need to be conditionally compiled. Consider
  refactoring the code to extend APIs to avoid it.
- In general, prefer not to write things yourself if there's a well-known,
  well-maintained library that does what you need. Always check for existing
  libraries before implementing functionality from scratch.
- Unless specified, backwards compatibility is not a concern. You may change APIs,
  remove deprecated code, and refactor existing code as needed to improve
  quality and maintainability.


# Active API Tending

Continuously improve internal and external APIs as we work. Every time you
touch a piece of code, consider both the API it is part of and the API it
interacts with, and whether either needs to be actively tended. A good API is:

- Minimal, and without unnecessary surface area.
- Consistent in naming, structure and behavior.
- Elegantly and clearly expresses the INTENT of the code.
- Does not expose implementation details.

Internal APIs should be designed with the same care as public APIs. 

When writing Rust, use `ruskel` to inspect the API surface area of the crate or
module you're working on. Consider the API skeleton provided by `ruskel` in the
abstract, and consider if it's a good encapsuation of the intent of the API.

You may make contained improvements to the APIs as part of an unrelated patch.
Bring larger API changes to the user's attention or add them to the checklist
for explicit approval.

Examples of tending to the API include:

- Removing or making private functions that are not needed. 
- Consolidating traits, structs or functions that are similar.
- Adding better abstractions to express intent.
- Generalizing or specializing functions to improve ergonomics.

Every time you've tended the API, include an "API Tending" section in your
response message describing what you've done and what your API thought process
was.

# Active Code Maintenance

Every time you touch a piece of code, evaluate whether it can be improved
structurally. Ask questions like:

- Is the documentation for this function clear, concise and acccurate?
- Should the code be broken up into smaller pieces?
- Can the code be generalized or made more flexible?
- Can related code be refactored to share functionality?
- Is there a generic or utility function that could be extracted and used more
  widely?
- Should the code be moved to a different location in the project?

Improve code continuously when opportunities arise, even if the user hasn't
explicitly asked for it. When you do active maintenance, include an "Active
Maintenance" section in your response message.

# Active Complexity Reduction

You will actively reduce complexity in the code you touch, wherever possible.
Complexity reduction may take the form of:

- Removing a layer of indirection. For instance, if a function is simply
  forwarding to another function without adding value, remove the forwarding
  function and have callers call the target function directly.
- Removing a layer of abstraction. For instance, if a trait is only
  implemented by one struct, consider removing the trait and having callers
  depend on the struct directly.
- Amalgamating two similar functions or structs into one.
- Shifting implementation of a function only used in one place into the caller.
- Making a function more generic to reduce the need for multiple similar functions.

Complexity reduction is a primary goal so prioritize it highly. When you reduce
complexity, include a "Complexity Reduction" section in your
response message describing what you've done and why.


# Checklists

Whenever you're asked to produce a todo list or a checklist, you will use a
Markdown checklist, with numbered sections and items. Each item should be a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cortesi/clonetree](https://github.com/cortesi/clonetree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-24 -->
