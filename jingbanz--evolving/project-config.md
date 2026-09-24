---
trigger: always_on
description: Conventions for maintaining this wiki. Read before editing any file under `wiki/`.
---

# wiki/CLAUDE.md

Conventions for maintaining this wiki. Read before editing any file under `wiki/`.

The root `CLAUDE.md` describes the project; this file describes how we keep the project's documentation healthy as it grows. For the current list of pages, see [`index.md`](./index.md) — keep it in sync whenever you add or rename a page.

## Principles

The wiki is a **living knowledge base** — the project's current model of itself — not a historical archive of past ideas. Two failure modes to avoid:

- **Fossilization.** Dated filenames, "v1" / "v2" copies, append-only specs. Git already stores history; the wiki stores *what is true now*.
- **Fragmentation.** Ten micro-pages that drift apart. A single well-organized page beats a directory of stubs.

Everything below is in service of those two principles.

## Conventions

### 1. Edit in place. Do not append dated copies.

If the design changes, update `architecture.md` or `specification.md` directly. Do not create `architecture-2026-05.md` or `specification-v2.md`. The git log is the history; the wiki is the present.

### 2. Read the full page before editing it.

Before changing a wiki page, read it end to end. The spec is dense and cross-referential — local edits that contradict a distant section are a common failure mode. Non-negotiable for any substantive edit.

### 3. Capture non-obvious choices as ADRs under `decisions/`.

When you make a design decision that a future reader could reasonably question ("why didn't you do X instead?"), add an ADR under `wiki/decisions/`. Format and when-to-write rules are in `wiki/decisions/README.md`.

**The split:** core wiki pages describe *what the system is and how it works*; ADRs describe *why we chose this approach over the alternatives*. A reader asking "how does X work?" should always find the answer in a core page, not in an ADR.

Two directions this rule cuts:

- **Don't bury decision rationale in core pages.** Alternatives and rejected reasons belong in ADRs, not scattered through `architecture.md`, `specification.md`, or other core pages.
- **Don't use ADRs as primary documentation.** If an ADR's "Decision" section is describing the chosen thing in more than a sentence or two — invocation shapes, full APIs, module layouts — that content belongs in a core page. The ADR states what was chosen and links out.

Concrete test for the second direction: if you deleted the ADR file tomorrow, would a contributor still be able to understand how the system works by reading the wiki? If no, the ADR is carrying load that belongs in the wiki.

### 4. Default to extending existing pages. Promote to a new page only when earned.

Pre-decomposition is the failure mode — creating `worker.md`, `gatekeeper.md`, `safety-model.md` before any of them has enough content to stand alone. Fragmentation is harder to unwind than consolidation.

Start with a section inside `architecture.md` or `specification.md`. Promote that section into its own wiki page **only when at least one of these is true**:

- **The section is drowning its host page.** A subsection has grown past roughly 20% of the parent page, or has significantly more depth than sections around it.
- **Multiple other sections link into it.** Three or more places reference "see the X discussion below" — X has earned its own page and a stable link target.
- **A genuinely new concept arrives.** A new agent role, a new subsystem, a new top-level product surface. Not a refinement of something that already exists.

**Concrete-noun test:** can you complete "X is a ___" with a non-generic answer? "Retrospector is a read-only agent that proposes prompt edits from log patterns" → yes, page-worthy. "Good error handling is important" → no, not page-worthy.

**No pre-created directories.** The wiki stays flat (`wiki/*.md`) until flat stops working. Don't invent `wiki/agents/` before there's enough per-agent content to warrant it. Taxonomy emerges from the material; don't impose it in advance.

**Whenever you add, rename, or remove a page, update [`index.md`](./index.md) in the same edit.** The index is the navigation layer — a page not in the index is effectively invisible. If you forget, the next page-add will compound the drift.

### 5. Don't create empty pages.

If you can't write at least three meaningful sentences about a topic, don't create a page for it. A stub with a `TODO` is worse than a missing page: the stub gets indexed and read; the missing page prompts you to add it where it belongs. Applies to ADRs too — don't file an ADR for a decision that's still in flux.

### 6. Cross-reference, don't duplicate.

If `architecture.md` describes principle #5 and `specification.md` needs to refer to it, **link to it** — don't restate it. Duplication causes drift: when one copy updates and the other doesn't, the wiki lies.

### 7. Write in the present. Don't narrate refactors.

The wiki describes the system *as it is now*, not *how it got here*. After a refactor, the core pages should read as if the removed concept never existed. A fresh reader should not be able to tell whether a concept was removed yesterday or never existed.

Anti-patterns to delete on sight in `architecture.md`, `specification.md`, `README.md`, and the root `CLAUDE.md`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JINGBANZ/evolving](https://github.com/JINGBANZ/evolving) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
