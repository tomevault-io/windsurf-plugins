---
trigger: always_on
description: > **ቆንጆ** — Beautiful. **根性** — Fighting spirit. **康宙** — Health of the universe.
---

# copilot-instructions.md — Konjo AI Project Conventions & Collaboration Guidelines

> **ቆንጆ** — Beautiful. **根性** — Fighting spirit. **康宙** — Health of the universe.
> *Make it konjo — build, ship, repeat.*

This file defines standing instructions for all AI and human contributors working on projects in this repository. Read it fully before writing, modifying, or deleting any code or documentation. These are not suggestions.

---

### 🌌 The Konjo Way (Operating in Konjo Mode)
"Konjo Mode" is a universal operating frequency applicable to any challenge, project, or interaction. It is the refusal to accept the mediocre, built on three cross-cultural pillars:

* **The Drive (根性 - Japanese):** Relentless fighting spirit, grit, and determination. Approaching impossible problems with boldness and never surrendering to the "standard way" when a harder, superior path exists.
* **The Output (ቆንጆ - Ethiopian):** Executing with absolute beauty and nobility. This requires *Yilugnta*—acting in a selfless, magnanimous, and incorruptible fashion for the ultimate good of the project—and *Sene Magber*—the social grace of doing things gracefully, respectfully, and beautifully.
* **The Impact (康宙 - Chinese):** Cultivating the "Health of the Universe" by building systems that are highly efficient, healthy, and in tune with their environments. It means eliminating waste, reducing bloat, and leaving the architecture fundamentally healthier than you found it.

---

## 🗂️ Planning First

- **Always read `PLAN.md` at the repo root before starting any task.** There is no `docs/planning/PLAN.md`; the authoritative roadmap is `/PLAN.md`. Current active version: **v3.6.0** (all 16 phases complete).
- Identify the relevant phase or milestone before writing or modifying any code.
- After completing work, update `PLAN.md`, `CHANGELOG.md`, and `README.md` to reflect what changed, what is done, and what is next.
- If a task deviates from the current plan, call it out explicitly before continuing.
- If any ambiguity exists, ask a single focused clarifying question before implementation. Do not ask multiple questions at once.

---

## 📁 File & Project Structure & Repo Health

**System Health is Mandatory (康宙).** A cluttered repository slows down human and AI compute. You must proactively suggest organizing files, grouping related modules into new directories, and keeping the root directory pristine.

**Propose Before Moving.** If you notice a directory becoming a junk drawer, propose a new taxonomy and confirm it with the user before executing bulk file moves.

**Continuous Cleanup.** Delete dead code immediately. Do not comment it out and leave it — use version control for history.

**No Graveyards.** Prototype code that is not being promoted must be deleted after the experiment concludes. Do not let the experiments/ or research/ directories rot.

**Naming Conventions:** New modules, crates, or packages must match the established naming conventions strictly.

---

## 🧱 Code Quality & Architecture

- **Shatter the box.** We are solving problems that have not been solved before. Do not reach for the nearest familiar pattern or standard library if it compromises efficiency.
- **Code must punch, kick, and break through barriers.** Clever code is not just welcome—it is required when it achieves leaps in performance. Correctness without elegance is a missed opportunity.
- **Extreme Efficiency is mandatory.** Every architecture decision must minimize resource usage: less CPU, less RAM, less disk space, less compute for training, and faster inference. Treat resource optimization as a core design discipline.
- **No Hallucinated Abstractions.** "Novel" does not mean "fake." When inventing new sub-transformer layers, quantization schemes, or memory management systems, do not hallucinate APIs or rely on "magic" functions. Ground your innovations in explicit tensor operations, raw mathematical formulations, and supported framework primitives.
- **All written code must be production-grade at all times.** No placeholders, no "good enough for now," no TODOs left in shipped code.
- Avoid code duplication. Extract shared logic into reusable utilities or modules.
- Add inline comments only where intent is non-obvious. When implementing a novel algorithm, write the math — don't hide it.

---

## 🔄 Version Control & Documentation Sync
* **Documentation is mandatory per prompt cycle**: Every prompt must result in updated documentation reflecting the current state of the system, including successes, failures, partial progress, blockers, and decisions. This is not gated by Ship Gate results or test outcomes.
* **Commit + Push on full success**: If and only if all Ship Gate conditions pass with zero violations, the system must automatically:
  * commit all changes with a clear, descriptive message
  * push to the remote repository immediately
* **No commit on failure**: If any Ship Gate condition fails, do not commit or push changes under any circumstance.
* **Failure-state documentation still required**: Even when gates fail, documentation must still be updated to reflect:
  * what was attempted
  * what failed
  * root cause analysis (technical, not narrative)
  * next corrective step

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [konjoai/vectro](https://github.com/konjoai/vectro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
