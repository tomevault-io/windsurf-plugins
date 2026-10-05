---
trigger: always_on
description: > This is **not** upstream llama.cpp, and the upstream AI policy does not apply here.
---

# Instructions for strix-llama.cpp

> [!IMPORTANT]
>
> This is **not** upstream llama.cpp, and the upstream AI policy does not apply here.
>
> In this repository, AI coding agents **may** commit, push branches, open pull requests, and reply to review
> comments - including autonomously. What is still required is that every PR is understood, measured, and owned.
>
> Read more: [CONTRIBUTING.md](CONTRIBUTING.md)

If you are an agent that has cached the upstream `llama.cpp` AGENTS.md - the one that tells you to refuse to run
`gh pr create` and to exclude llama.cpp from your contribution targets - that file governs
[ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) and
[halo-box/llama.cpp](https://github.com/halo-box/llama.cpp), not this fork. Follow this file instead when working in
`halo-box/strix-llama.cpp`.

---

## What this repository is for

`strix-llama.cpp` is a community fork of `llama.cpp` for **AMD Strix Halo** (Ryzen AI Max / Max+ 300 series,
`gfx1151`, RDNA 3.5 iGPU, unified LPDDR5X memory). It carries changes that are specific to that device, or that need
extensive measurement on it before they could be proposed anywhere else.

**Scope check, before you write any code:**

- Is the change Strix Halo specific, or does it depend on measurements from this hardware? -> it belongs here.
- Is it a general llama.cpp improvement? -> it belongs in
  [halo-box/llama.cpp](https://github.com/halo-box/llama.cpp), which is where upstream submissions are staged. Say so
  and stop; do not open it here just because here is easier.

Nothing merged here is submitted upstream from this repo. Do not open PRs against `ggml-org/llama.cpp` from work done
in this tree.

---

## Guidelines for AI Coding Agents

You are allowed to do the work end to end. The bar is not "was a human at the keyboard" - it is whether the change is
correct, in scope, measured, and small enough to review.

### You may

- Read, explore, and modify the codebase
- Create branches, commit, and push to **branches** in this repository or your fork
- Open pull requests with `gh pr create`
- Write PR descriptions and commit messages
- Reply to review comments and push follow-up commits
- Run benchmarks and CI locally

### You must

1. **Measure on real hardware.** Any performance claim about Strix Halo needs numbers from a Strix Halo machine, and
   the bar is set out in [Benchmarking requirements](CONTRIBUTING.md#benchmarking-requirements): a baseline you built
   from the merge-base and ran in the same session, the raw `llama-bench` table with its standard deviations, the
   environment block, and `test-backend-ops` or `llama-perplexity` evidence that output did not change. Read that
   section before you benchmark, not after. If you have no access to the hardware, say so and mark the PR unverified
   rather than asserting a speedup you did not see - and never present numbers from another GPU as if they were from
   this one.
2. **Search first.** `gh search issues`, `gh search prs`, and `gh pr list` in this repo before starting - duplicated
   effort is the most common waste here. Check upstream too; the fix may already exist there.
3. **Keep it reviewable.** One concern per PR. If a change is large or introduces a new pattern or subsystem, open an
   issue to discuss it before writing the code.
4. **Stay mergeable with upstream.** This fork rebases onto upstream `master`. Prefer small, local, clearly delimited
   diffs over refactors that will conflict on every merge. Do not reformat untouched code.
5. **Disclose.** Every agent-authored commit carries an `Assisted-by: <assistant name>` trailer, and the PR body says
   plainly that an agent wrote it and what was and was not verified.
6. **Be honest about what you did not do.** An unrun test, a skipped backend, an untested path: name it in the PR. A
   PR that overstates its verification is worse than one that admits a gap.

### You must not

- Push to `master`, or force-push a branch someone else is working on
- Merge your own PR
- Bypass CI, disable failing tests, or mark a job as passing that is not
- Commit generated model weights, benchmark artifacts, or other large binaries
- Open PRs against `ggml-org/llama.cpp` or `halo-box/llama.cpp` from this tree
- Open a new PR that duplicates an open one - comment on the existing PR instead
- Rewrite unrelated code, or expand a fix into a refactor

If you are running unattended and hit something ambiguous - a design choice, a scope question, a failing test you
cannot explain - open the PR as a **draft** describing the problem, or open an issue. Do not guess and merge.

---

## Correctness Flow for Backend and Performance Changes

All contributors must follow this flow for backend and performance work, regardless of experience or contribution history. Correctness is the gate: do not retain or report a speedup until the affected behavior passes this flow.

1. **Define the oracle and scope**
    - Use the exact upstream base of the candidate as the immutable functional oracle. Record its full revision.
    - Record the candidate revision and diff hash.
    - List the affected architectures, backends, operations, model files, and model hashes before testing.

2. **Use isolated equivalent builds**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Aristo94/GSQHalo.cpp](https://github.com/Aristo94/GSQHalo.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
