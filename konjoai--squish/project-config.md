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

- **Always read `docs/planning/PLAN.md`, `ROADMAP.md`, or equivalent planning docs before starting any task.**
- Identify the relevant phase, step, or milestone before writing or modifying any code.
- If no plan exists, create one before proceeding and ask for confirmation.
- After completing work, update `PLAN.md`, `ROADMAP.md`, `README.md`, and any relevant docs to reflect what changed, what's done, and what's next.
- If a task deviates from the current plan, call it out explicitly before continuing.
- If any ambiguity exists in requirements, ask clarifying questions before implementation.

---

## 📁 File & Project Structure & Repo Health

**System Health is Mandatory (康宙).** A cluttered repository slows down human and AI compute. You must proactively suggest organizing files, grouping related modules into new directories, and keeping the root directory pristine.

**Propose Before Moving.** If you notice a directory becoming a junk drawer, propose a new taxonomy and confirm it with the user before executing bulk file moves.

**Continuous Cleanup.** Delete dead code immediately. Do not comment it out and leave it — use version control for history.

**No Graveyards.** Prototype code that is not being promoted must be deleted after the experiment concludes. The `experimental/` directory exists for research code awaiting validation — not for permanent storage. Each module in `experimental/` must have: (1) a concrete promotion criterion (a specific benchmark number it must hit) and (2) a named owner. If neither exists, the module is deleted immediately. The 90-day review clock starts when the promotion criterion is *written*, not when the file is moved. **squish/ must not grow above 100 active Python files.** Any addition requires a corresponding deletion or demotion to `experimental/`, or explicit written justification.

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
- Prefer removal over addition. Every new line of code must justify its existence. If a simpler, more efficient solution exists that requires fewer lines, it is the only acceptable solution.

---

## 🔄 Version Control & Documentation Sync

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [konjoai/squish](https://github.com/konjoai/squish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
