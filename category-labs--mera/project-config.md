---
trigger: always_on
description: Simplicity, elegance, and correctness are the bar for everything here — code, documentation, tests, and examples alike.
---

### Core Values

Simplicity, elegance, and correctness are the bar for everything here — code, documentation, tests, and examples alike.

- Correctness comes first: prefer the obviously correct implementation over a shorter or cleverer one.
- Write the simplest code that solves the problem; reach for a plainer construct over a clever one a reader has to decode.
- Treat elegance as what remains after removing the unnecessary — fewer moving parts and clearer names, not more abstraction.
- Write documentation in plain language: explain things directly, avoid jargon, and define a domain term only when the reader genuinely needs it.

### API Design Philosophy

This repository is a library. Its API is part of its documentation.

North star: by reading a function name and its parameters, a developer should be able to predict the implementation's behavior.

The library exposes unopinionated primitives ("Lego blocks") that consumers compose into their own flows. Sensible-default orchestrators may live alongside primitives only when the defaults are truly obvious and ecosystem-compatible; they sit beside the primitives rather than replacing or hiding them. If a default derivation path is not wallet-interoperable, keep it app-owned rather than baking it into core.

- Choose literal, self-descriptive names.
- Prefer explicit data structures over magic defaults.
- Keep return values predictable and stable.
- Make limits and failure behavior obvious in names, types, docs, or thrown errors.
- Do not add thin convenience layers that obscure ordering, length limits, revert behavior, or transport assumptions.
- Do not wrap dependency primitives unless the wrapper adds library-specific behavior, a stable package error boundary, or a meaningful format guarantee.
- Keep generic utilities out of the root SDK API unless callers need them to complete a documented library workflow. Wire-format helpers may remain internal when they provide a stable package error boundary or format guarantee.
- Prefer one canonical representation for raw bytes at public boundaries. Use `Uint8Array` unless a wire format or browser API boundary requires a different shape.
- Preserve runtime validation at cryptographic, string, wire, and untrusted JSON boundaries. Avoid "perfect validation" of typed internal invariants that TypeScript or a dependency already owns.
- Keep object-parameter APIs when the operation is likely to gain optional parameters or when named fields make security-sensitive inputs harder to mix up.

### Secret Handling

- Zero buffers the library allocated. Inputs belong to the caller, and so are the returned buffers.
- Treat zeroing as a bound on secret lifetime and reachability, not as erasure from the process. Documentation must not claim more.

### Documentation and Examples

- Documentation prose is technical writing held to a hard bar: simple, concise, and detailed at once. Every sentence must add information; cut sentences that only set up, restate, or editorialize on what adjacent sentences already show.
- Comments and JSDoc must add information beyond a symbol's name, type, and nearby code. Omit them when the code already states the contract.
- Prefer names that carry stable context over comments that explain it. When a comment remains useful after renaming, keep only the fact the name cannot express.
- Public SDK functions should have complete, accurate JSDoc.
- Use appropriate JSDoc tags to describe the API contract, return behavior, input constraints, observable side effects, and failure modes.
- Document security-sensitive behavior explicitly, especially for key material, randomness, WebAuthn prompts, encryption nonces, storage formats, and mutation/zeroing behavior.
- Document thrown `MeraError` codes with the appropriate JSDoc tag.
- Examples should be runnable, concise, and focused on library behavior, not on provider boilerplate.
- Give each demo a clear teaching goal. Keep only the code, UI, configuration, and documentation needed to teach or run that workflow.
- In examples, a value's meaning lives in a descriptive variable name (`recipient`, `privateKey`), never in a comment. Extract inline literals to named variables, delete comments that restate a name, and move provenance worth keeping into surrounding prose.
- The root README is a nontechnical project overview. Installation, examples, compatibility, security details, API documentation, and the demo live on the documentation website.
- Keep documentation prose neutral: name keys, secrets, and passkeys plainly ("the passkey", "one encrypted secret") rather than attributing them to the reader ("your passkey", "a secret you provide" / "you own").
- Internal helpers with non-obvious invariants should have short `//` comments or full JSDoc.
- Document observable behavior, not caller instructions: state what a function does to its inputs and outputs rather than what the caller may or should do with them. Callers derive correct usage from the stated facts.
- Omit behavior a reader assumes by default and facts a reader cannot act on: not modifying inputs, internal copies, internal zeroing. Silence implies the default; document the exceptions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [category-labs/mera](https://github.com/category-labs/mera) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
