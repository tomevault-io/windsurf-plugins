---
trigger: always_on
description: **Always apply this when creating or modifying any content under `docs/site/`.** It is the Spacedock adaptation of Recce's documentation-writing standard ([`writing-content/reference/doc.md`](https://github.com/DataRecce/recce-team/blob/main/recce-team/skills/writing-content/reference/doc.md)). It governs both the **shape** of a page (structure and simplicity) and its **voice** (terminology, register, and the tells to avoid). When a question falls outside it, fall back to the `elements-of-style:
---

# Authoring directive for `docs/site/`

**Always apply this when creating or modifying any content under `docs/site/`.** It is the Spacedock adaptation of Recce's documentation-writing standard ([`writing-content/reference/doc.md`](https://github.com/DataRecce/recce-team/blob/main/recce-team/skills/writing-content/reference/doc.md)). It governs both the **shape** of a page (structure and simplicity) and its **voice** (terminology, register, and the tells to avoid). When a question falls outside it, fall back to the `elements-of-style:writing-clearly-and-concisely` (Strunk) skill.

## The first rule: simple beats complete

Spacedock is not complicated; wordy docs make it *feel* complicated. Every edit should make the reader's job smaller, not bigger.

- **Lead with the problem or the payoff, not the definition.** Open each page with why the reader is here and what they will be able to do, then name the mechanism.
  - ✗ "Spacedock is a multi-agent orchestrator."
  - ✓ "You have work that needs doing in stages, with a human sign-off before anything ships. Spacedock runs that for you."
- **Introduce the fewest terms possible, as late as possible.** Don't front-load a glossary. Define a term on first real use, gloss it once, then just use it. If a page introduces more than a handful of new terms, cut or defer some.
- **Lead with what the user sees and must know; keep the how-it-works light.** Name the visible behavior and the required concepts first. Internal mechanics (scheduling and reuse conditions, file/branch naming templates, parser internals, query plumbing) get at most a sentence or a link to the source. If a paragraph reads as protocol documentation, compress it or cut it.
- **A paragraph that cannot be restated as one verifiable claim is flowchart.** Behavior checked against the owning skill compresses to a single stated fact; phase-by-phase narration of a skill's internals cuts entirely and the page loses nothing.
- **Product anatomy is not a reader concept.** Do not name internal components (launcher, plugin, host) unless the reader must type or choose between them. The reader installs Spacedock and launches Spacedock; how it decomposes is not their problem.
- **Write from the reader's seat, never the maintainer's.** If a sentence's subject is the build, the script, or the parser, recast it around what the reader gets or does.
  - ✗ "The build emits a curated `llms.txt` index at the site root."
  - ✓ "Start from `llms.txt`, the curated index of these pages."
- **A well-named command needs no caption.** "Run `spacedock doctor`." is complete; explaining that it diagnoses problems repeats the name. Likewise, never pre-document what a tool prints interactively (the install script already prints its own `PATH` note).
- **A heading is a promise; the section delivers exactly that.** Codex setup is not "Install Spacedock"; a tab the reader chose ("no Homebrew") must not explain the thing they opted out of. Headings also take the reader's angle: name what the reader does or gets ("Turn the report into a workflow"), not what the product emits ("The commission offer").
- **Serve the typical reader; route edge audiences elsewhere.** A Get-started page carries only the path a typical user walks. Contributor and from-source material lives in the repo, not inline, not even as one sentence.
- **Document the durable value, not the output inventory.** Do not enumerate an artifact's current sections or fields; they drift with releases. Name what the reader gets out of it (the survey report is "the four things you learn", not a list of its sections).
- **A real example with sample output beats description.** One concrete command plus the output the reader will see teaches faster than paragraphs. Compose samples from the product's real templates, and never restate in prose what the sample already says ("Accept this design, or tell me what to change" makes a "nothing is generated until you accept" sentence dead weight).
- **Keep agent-facing surfaces off user pages.** Commands meant for the agents (`spacedock status`, `spacedock dispatch`) are not taught in Get started, even though their output is real.
- **The docs may defer to the agent.** Spacedock runs inside a coding agent; "ask the agent if anything is unclear" is a legitimate close. Pages cover what the reader needs before and between sessions, not every contingency within one.
- **For an agent-operated feature, the page's job is trust, then deferral.** State the guarantees that make handing over safe (nothing is auto-replaced; it waits for your approval), then defer the mechanism to the agent. Detail beyond the guarantee is reassurance theater.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spacedock-dev/spacedock](https://github.com/spacedock-dev/spacedock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
