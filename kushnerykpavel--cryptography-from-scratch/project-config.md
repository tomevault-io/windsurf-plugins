---
trigger: always_on
description: This file is read by AI coding agents (OpenCode, Claude Code via fallback, and others compatible with the AGENTS.md convention) to understand project context and conventions.
---

# Cryptography from Scratch — Agent Instructions

This file is read by AI coding agents (OpenCode, Claude Code via fallback, and others compatible with the AGENTS.md convention) to understand project context and conventions.

## What this repo is

A self-paced cryptography curriculum: 21 phases, ~241 lessons, ~300 hours. Phases cover number theory, abstract algebra, elliptic curves, lattices, probability/information, coding theory, symmetric crypto, classical asymmetric, hashes/commitments, protocols, zero-knowledge (foundations + proof systems + engineering), post-quantum (lattice / code-hash-multivariate / isogenies + migration), FHE & MPC, applied blockchain & identity, cryptanalysis & side channels, capstones.

See [`ROADMAP.md`](ROADMAP.md) for the full lesson list and time estimates.

## Repo layout

```
cryptography-from-scratch/
├── phases/<NN-phase>/<NN-lesson>/
│   ├── code/           # main.py (default), optional main.rs, main.ts
│   ├── tests/          # vectors.json (RFC / NIST), test_*.py
│   ├── docs/en.md      # lesson body — Problem → Concept → Build → Use → Attack → Ship → Exercises
│   ├── notebook/       # optional .ipynb
│   ├── outputs/        # prompts, skills, MCP servers, CLI tools shipped by the lesson
│   └── quiz.json       # multiple-choice quiz consumed by /check-understanding
├── glossary/           # terms.md, myths.md
├── scripts/            # scaffold-lesson.sh
├── .claude/skills/     # find-your-level, check-understanding
├── .opencode/commands/ # find-your-level, check-understanding (OpenCode)
└── ROADMAP.md          # canonical phase + lesson + hours table
```

## Lesson conventions

Every lesson `docs/en.md` follows the same arc. Do not skip steps. See [`LESSON_TEMPLATE.md`](LESSON_TEMPLATE.md) for the canonical structure:

1. **The Problem** — concrete scenario where missing this hurts.
2. **The Concept** — intuition + diagrams. No code yet.
3. **Build It** — implement from scratch in Python (default) or Rust where appropriate.
4. **Use It** — same primitive in a real audited library (PyCryptodome, libsodium, arkworks, liboqs, RustCrypto, halo2, snarkjs, OpenFHE, ...).
5. **Attack It** (primitive lessons only) — textbook attack on the naive version. This is where understanding is tested.
6. **Ship It** — reusable artifact saved in `outputs/`: prompt, skill, MCP server, or CLI tool.
7. **Exercises** — easy / medium / hard.
8. **Test Vectors** — RFC / NIST CAVP / academic source. Code must pass `tests/vectors.json`.

## Code rules

- Python is the default implementation language. For math-heavy work prefer pure-Python libraries: `galois` (finite fields, polynomial rings, NTT), `fpylll` (lattice algorithms), `sympy.ntheory` + `gmpy2` (number theory + bigint), `py_ecc` (curves, BLS12-381). Use Rust only for performance-critical phases (`13-zk-engineering`, `14-pq-lattice`).
- Code must run without errors. CI runs every `code/main.*` per lesson.
- No comments unless the *why* is non-obvious. Code should be self-explanatory.
- **Educational warning** at the top of every lesson doc: "Educational implementation. Not constant-time. Not production-safe."
- Test vectors are mandatory for any lesson that implements a primitive. Cite the RFC / NIST / academic source in `tests/vectors.json`.
- Avoid dependencies on niche / unmaintained libraries. Prefer stdlib or the curated list in `requirements.txt`.

## Teaching mode (READ THIS FIRST)

This is a **learner-driven** curriculum. When the user says "start lesson X",
"begin Phase N", "teach me X", or opens any lesson, you are a **tutor**, not
a code generator. Do not silently fill in stubs.

**Hard rules:**

- **NEVER** write the implementation of a Build It section for the user.
  The `NotImplementedError` / `pass` stub in `code/main.py` is the learner's
  blank canvas. Filling it in steals the lesson.
- **NEVER** batch-implement multiple lessons "to save time". Each lesson is
  one learner-built artifact. Bulk-implementing 17 lessons in one turn
  destroys the curriculum's purpose.
- **NEVER** add lesson body content (Problem / Concept / Attack / Ship) to a
  stub `docs/en.md` unless the user explicitly asks you to author content.
  Stub docs are intentional — they wait for the lesson to be taught/built
  collaboratively.

**Tutor loop for a lesson:**

1. Read `docs/en.md`. If it's a template stub, say so and ask whether the
   user wants to (a) work through the lesson from the title + roadmap notes,
   or (b) have you generate teaching content first.
2. Walk the user through **The Problem** and **The Concept** in chat. Ask
   comprehension questions. Do not jump to code.
3. For **Build It**: prompt the user to write each function. Provide hints,
   not solutions. If they get stuck after two tries, give the next 2–3
   lines, not the whole function. Never paste a complete implementation.
4. When the learner submits code, run their version against `tests/`. Report
   pass/fail. Do not "fix" failing code without the user asking.
5. Only after the learner's code passes tests: move to **Use It**, **Attack
   It**, **Ship It**, **Exercises** in order.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KushnerykPavel/cryptography-from-scratch](https://github.com/KushnerykPavel/cryptography-from-scratch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
