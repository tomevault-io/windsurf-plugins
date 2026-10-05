---
trigger: always_on
description: Project context for Claude Code. Read this whole file before any action.
---

# bend-ml — Machine Learning in Bend 2

Project context for Claude Code. Read this whole file before any action.

## Why this project exists

On 2026-09-29, Victor Taelin (creator of Bend/HVM, company HOC) published an "info dump" about the future of Bend:

- HOC is raising a **US$ 20M** round and will set up two internal teams: **BendCore** (theory, compiler, ecosystem, adoption) and **SupGen** (symbolic AI).
- The ecosystem wishlist explicitly includes: **"AI framework / PyTorch / llama.bend / etc."**, besides "all good parts of mathlib, npm, hackage".
- They launched **BendHub**: named, versioned libraries, with a "mini Hacker News" on the site to discover, vote on and discuss packages.
- Current adoption: ~400 non-cloud IPs per day using the `bend` command. He himself thinks it is low.
- Bend's thesis: **"a language evolved by AI, secured by math"**. AI writes the code, LAWS (proofs verified by the kernel) guarantee correctness.

When asked what someone who wants to join one of the teams should do, he answered:

> build something in Bend so 1. VCs see traction and decide to invest in us 2. send that thing to impress us and be hired

Link: https://x.com/VictorTaelin/status/2104956654542639176

**Dual goal of this project:**

1. **Measurable traction**: packages on BendHub that other people use (dependents, upvotes, downloads).
2. **Portfolio**: something impressive enough to send to Taelin in an application.

**Chosen angle:** the differentiator only Bend has is **dependent types**. The central proposal is a "PyTorch where a shape error does not compile", using AI to write and LAWS to guarantee, that is, dogfooding Bend's own thesis.

## About the author

- Renan (nuxyel), backend developer. Main language: Python. Also TypeScript/Node.
- Currently studying ML formally (Stanford/DeepLearning.AI ML Specialization). Practical experience with NLP (spaCy, stanza).
- **No background in type theory or formal proofs.** When writing proofs, explain the reasoning in plain language in comments and in chat. The project is also a learning project.
- Machine: Core Ultra 7 155H, 32 GB RAM, **RTX 4050 (6 GB VRAM)**, Windows/Linux. Keep the VRAM limit in mind in any GPU demo.

## Working rules

1. **Bend 2 is new and changes fast.** Its syntax is different from Bend 1's (which was Python-style). Do not assume syntax from memory. Before writing code, read the official documentation, the examples and the Base library. When in doubt, read the source code.
2. **Pin the Bend version** used by the project and record it in `NOTES.md`.
3. **Run the checker all the time.** It is fast; use it as a feedback loop on every change.
4. **Never work around a proof.** If Bend has some mechanism equivalent to `sorry`/axiom/assume, do not use it without warning explicitly and recording it in `NOTES.md` as debt. A false or unproved LAW destroys the project's value.
5. **Floats are not reals.** Do not try to prove numerical properties about floating point. Prove what is structural (shapes, types, composition) and validate the numerics with tests (gradient checking, comparison with a Python reference).
6. **Python references** live in `reference/` (PyTorch, tiktoken, etc.) and only serve to generate expected values for tests.
7. Small, frequent commits, with clear messages.
8. Every proved LAW must appear in the corresponding package's README, in human language.
9. **Everything in the repository is written in English**: code comments, documentation and commit messages.
10. **No attribution lines in commits or pull requests**: no `Co-Authored-By` trailers and no "Generated with" footers.

## Phase 0 — Reconnaissance (do first, before any code)

Investigate and record everything in `NOTES.md`:

- [ ] Installed Bend version and how to update it.
- [ ] Where the documentation, the examples and the Base library are.
- [ ] What Base already offers: Nat, List, String, Map, arrays. **Which lemmas already exist** (Base is known to have few or no Nat lemmas).
- [ ] Number support: integers (U32/I32/U64?), **floats (F32/F64?)** and the available operations.
- [ ] Available compilation backends (JS, C, GPU). **Which GPU backend runs on an RTX 4050** (CUDA? Metal only?). That decides whether Phase 3 runs on GPU or CPU.
- [ ] State of IO / C interop: reading files is necessary to load datasets and weights.
- [ ] How to publish on BendHub (namespace, versioning, package format).
- [ ] **Whether BendHub already has** a tokenizer, a tensor library or an autograd. If it does, evaluate whether it is better to contribute or to differentiate.

Expected output: `NOTES.md` with objective answers and a recommendation on adjustments to the plan below.

## Phase 1 — `bpe.bend` (target: a few days)

Byte-level BPE tokenizer (GPT-2 style: a base vocabulary of 256 bytes + merges).

**Scope:**
- `train(corpus, n_merges) -> MergeTable`
- `encode(table, bytes) -> List<Token>`
- `decode(table, tokens) -> bytes`

**LAWS to prove:**
- **Roundtrip:** for any well-formed merge table and any byte sequence, `decode(encode(s)) == s`.
- **Vocabulary bound:** every token emitted by `encode` is smaller than the table's vocabulary size.
- (If feasible) `decode` distributes over concatenation of token lists.

**Tests:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nuxyel/bend-ml](https://github.com/nuxyel/bend-ml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
