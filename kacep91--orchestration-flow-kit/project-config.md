---
trigger: always_on
description: - Solve the requested problem with the smallest complete change. Apply KISS,
---

# Global Guardrails

- Solve the requested problem with the smallest complete change. Apply KISS,
  DRY, Occam's razor, and YAGNI: reuse repository patterns and native APIs before
  adding local code, and never add a dependency for a small straightforward job.
- Preserve unrelated and foreign changes. Never revert work you did not make.
- Do not weaken validation, data protection, security, accessibility, or an
  explicitly requested behavior. Avoid speculative cleanup and abstractions.
- Do not investigate unprompted long-running edge cases, write excessive tests,
  or dive deeply into security analysis. Do so only when explicitly requested.
- Never compute, require, or mention SHA-256 values for files, content, review,
  handoff, or verification; that depth is unnecessary unless explicitly requested.
- Do not stage, commit, push, install dependencies, edit lockfiles, or change
  infrastructure unless the user explicitly requests it.
- Read the root and applicable descendant `AGENTS.md` files before touching a
  path. Load a matching skill before the work it governs.
- Prefer `rg` for discovery, structured parsers for structured data, and
  `apply_patch` for manual file edits.

# Planning

- Sol orchestrates the task. Before any code, test, or configuration change, or
  any bug investigation, require a `.tasks/<slug>/plan.md` with assumptions,
  success criteria, scope, validation, and detailed checkboxes. A simple
  read-only lookup is exempt.
- Keep the plan current as work is validated. Do not bypass its scope or gates.
- For a large or risky task, first write a short design draft in memory. Send
  exactly one packet-first technical pre-mortem to `premortem-reviewer` on Sol
  Medium before writing the final plan. The packet must contain the draft,
  scope, assumptions, ownership, risks, and proposed validation.
- A pre-mortem `PASS` permits writing the final plan; implementation starts only
  after that plan exists. `REVISE` requires correcting the plan before
  implementation and does not trigger another pre-mortem in the
  same session. `BLOCK` stops the task until the blocker is resolved or the
  user explicitly changes the request. Use one pre-mortem dispatch per session
  unless the user explicitly asks for another.

# Roles

- Sol orchestrates from bounded Luna evidence. By default, the Luna worker that
  owns a behavior package also performs the bounded discovery needed to
  implement and test it. Use a separate `explorer` or `docs-reader` only when
  research is independently parallelizable or evidence is required to define
  package boundaries before assigning a writer. Do not pay for duplicate
  repository discovery. Sol minimally verifies the final diff and critical claims.
- Use `luna-worker` by default for routine bounded implementations. It is Luna
  Max. Use `luna-fast-worker` for tightly bounded mechanical or trivial changes
  that do not require broad discovery or fragile reasoning. Implementation and
  test execution stay on Luna; Sol plans, replans, and orchestrates. Assign one
  writer to each changed file in a wave.
- Use `browser-tester` for browser diagnostics or visual validation; it loads
  `playwriter` and must not save or submit persistent application data.
- The main thread maintains the review packet with
  `skills/handoff/scripts/update_review_packet.py`; do not
  spawn a model merely to merge packet sections or embed the current diff.
- Use `simple-code-reviewer` for the gated simple review and `code-reviewer` for
  ordinary or strong review. Both follow the code-reviewer skill and report
  findings only. `simple-code-reviewer` is Sol Low; `code-reviewer` is Sol High.
- Subagents do not delegate recursively.

# Execution

- Organize work into cohesive behavior packages and explicit waves. Parallelize
  only genuinely disjoint packages with disjoint file ownership. Never give two
  writers the same file in one wave.
- The main thread owns sequencing, integration, conflict resolution, final
  verification, and the user-facing result. Inspect agent claims against the
  repository instead of trusting summaries blindly.
- Close completed agents before later waves where the runtime supports it.
- Every implementation worker reads applicable guidance and surrounding code,
  edits only its allowlist, runs focused validation, inspects its diff, and fixes
  failures caused by its own work.
- Render and dispatch one worker package with the [worker-packet CLI](skills/writing-plans/scripts/render_worker_packet.py),
  not the full plan or unrelated wave/ledger data. A same-owner retry sends only
  its rendered retry delta; the initial package and contracts remain authoritative
  and are not resent or replaced.
- Role TOML prompts stay stable. Task-local objective, scope, contracts, steps,
  evidence, and validation belong in the rendered packet, not in a role prompt.
- Worker handoffs must fit within 1500 characters. Include only changed files,
  delivered behavior, exact validation results, task-owned failure state,
  assumptions, and residual risk; do not retell the plan or diff.
- Run repository-supported focused checks for changed files. Re-run a passed
  check only when relevant files changed afterward or a failure needs
  confirmation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kacep91/orchestration-flow-kit](https://github.com/Kacep91/orchestration-flow-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
