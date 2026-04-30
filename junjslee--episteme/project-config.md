---
trigger: always_on
description: > Operational contract for any AI coding agent working inside this repository.
---

# AGENTS.md — episteme

> Operational contract for any AI coding agent working inside this repository.
> Human-facing documentation: [`README.md`](./README.md). Durable
> first-principles: [`kernel/`](./kernel/).

If you have 500 tokens, load [`kernel/SUMMARY.md`](./kernel/SUMMARY.md).
Everything below is the operational contract; it does not replace the kernel.

---

## What this repository is

A portable cognitive kernel for AI agents. The kernel is markdown
(vendor-neutral). Adapters mount the kernel into specific runtimes
(Claude Code, Hermes, future). The kernel defines *how the agent thinks*;
everything else in this repo is delivery plumbing.

**Do not treat this repo as a general-purpose codebase.** It is a
governance + cognition product whose own artifacts are its thesis.

---

## Repository map (what lives where)

```
kernel/          philosophy; markdown; vendor-neutral; the contract
  SUMMARY.md     load first (30-line distillation)
  CONSTITUTION.md    root claim, four principles, nine failure modes (+2 planned for v1.0 RC: framework-as-Doxa, cascade-theater)
  REASONING_SURFACE.md   Knowns/Unknowns/Assumptions/Disconfirmation
  FAILURE_MODES.md       named modes ↔ counter artifacts
  OPERATOR_PROFILE_SCHEMA.md  how operators encode their worldview
  KERNEL_LIMITS.md       when this kernel is the wrong tool
  REFERENCES.md          attribution for every load-bearing borrow
  CHANGELOG.md           versioned kernel history
  HOOKS_MAP.md           kernel invariants ↔ runtime hooks
  MANIFEST.sha256        kernel integrity digest

demos/           reference deliverables produced by the loop itself
  01_attribution-audit/  canonical four-artifact shape; start here

core/
  memory/global/    operator's personal memory (gitignored; do NOT write)
  hooks/            deterministic safety + workflow hooks
  harnesses/        per-project-type operating environments
  schemas/          memory + evolution contract JSON schemas
  adapters/         adapter target configurations
  agents/           subagent persona definitions

adapters/claude/  Claude Code delivery layer
adapters/hermes/  Hermes (OMO) delivery layer
skills/           reusable operator skills (custom/vendor/private)
templates/        project scaffolds, example answer files
docs/             architecture, contracts, setup guides
src/episteme/    CLI + core library
tests/
```

---

## Build, test, sync

```bash
# Install (idempotent)
pip install -e .

# Verify health
episteme doctor

# Verify kernel integrity
episteme kernel verify   # detects drift in managed files

# Push kernel + profile to all adapters
episteme sync

# Run tests
PYTHONPATH=. pytest -q

# Static check
python3 -m py_compile src/episteme/cli.py
```

Local Python work runs in whichever Python invokes the CLI (`sys.executable`). Pin a specific runtime via `$EPISTEME_PYTHON_PREFIX` (install root) or `$EPISTEME_PYTHON` (exact binary). Set `EPISTEME_REQUIRE_CONDA=1` to enforce Conda `base`.

---

## Kernel invariants (do NOT modify without the Evolution Contract)

1. The four principles in `kernel/CONSTITUTION.md` are load-bearing. Adding, removing, or reframing one requires a major version bump in `kernel/CHANGELOG.md` and a propose → critique → gate → promote loop per `docs/EVOLUTION_CONTRACT.md`.
2. The six-failure-mode taxonomy in `kernel/FAILURE_MODES.md` is a 1:1 mapping. Removing a counter artifact means naming which failure mode is now unprotected. If the answer is "none" — the artifact was not earning its place.
3. The Reasoning Surface is four fields: Knowns, Unknowns, Assumptions, Disconfirmation. Do not rename or collapse them.
4. `kernel/KERNEL_LIMITS.md` declares the kernel's boundary. Claims to universal applicability without updating this file violate Principle I.
5. `kernel/REFERENCES.md` is the attribution contract. Introducing a new load-bearing concept into kernel wording without a primary-source entry violates Principle I.

---

## Workflow convention for non-trivial work

All consequential edits follow the kernel's own loop:

1. **Frame.** State the Core Question in one sentence. Identify the uncomfortable friction driving the work.
2. **Decompose.** Fill the Reasoning Surface (Knowns / Unknowns / Assumptions / Disconfirmation). For high-impact work, provide 2+ options with trade-offs and an explicit because-chain.
3. **Execute.** Prefer smallest reversible action that produces new information. One bounded lane per task owner.
4. **Verify.** Validate against success criteria, not effort. Re-check each assumption. Evaluate hypothesis: validated / refined / invalidated.
5. **Handoff.** Update `docs/PROGRESS.md`, `docs/NEXT_STEPS.md`. Name residuals explicitly.

High-impact decisions must record to `.episteme/reasoning-surface.json` before the action. See `kernel/HOOKS_MAP.md`.

---

## Boundaries

### Do NOT touch

- `core/memory/global/*.md` — operator's personal memory; gitignored; writing to it is an identity violation
- `.claude/settings.local.json` — machine-local overrides; gitignored
- `kernel/MANIFEST.sha256` directly — regenerate with `episteme kernel update` after intentional kernel edits
- Any file matching `**/.env*`, `secrets/*`, private keys

### Handle with care (checkpoint before acting)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [junjslee/episteme](https://github.com/junjslee/episteme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
