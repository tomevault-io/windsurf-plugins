---
trigger: always_on
description: **How to use this document.** Sections 0 through 8 are the master brief. Paste it once at the start of the project and write it to `CLAUDE.md` at the repo root so it persists across sessions. Then run Batches 1 through 6 one at a time, in order, pasting each batch as its own instruction. Do not paste all six batches at once. Akil's own build rule caps a build list at roughly 15 items because longer lists produce silent errors, and each batch here is already at or near that ceiling.
---

# AkilBello.com v4.0: Claude Code Build Prompt

**How to use this document.** Sections 0 through 8 are the master brief. Paste it once at the start of the project and write it to `CLAUDE.md` at the repo root so it persists across sessions. Then run Batches 1 through 6 one at a time, in order, pasting each batch as its own instruction. Do not paste all six batches at once. Akil's own build rule caps a build list at roughly 15 items because longer lists produce silent errors, and each batch here is already at or near that ceiling.

**Before anything else, read Section 8.** The client folder is mounted in this project. It is about 5 GB and most of it is not website material: old builds, a 2.5 GB video library, research corpora, analytics exports, and one file of personal subscriber data that must never be opened. Section 8 says exactly which files to read, which to mine for assets, and which to leave alone. Do not explore the folder on your own.

Everything in Sections 1 through 7 is locked. Everything in the batches is locked unless a line says otherwise. Where something is genuinely undecided it is marked **RAISE, DO NOT GUESS**, which means stop and ask Akil rather than picking something reasonable.

---

## 0. Project Brief

Build v4.0 of akilbello.com, the personal and professional site of Akil Bello: standardized testing expert, writer, and speaker. Director of College Advising and FAFSA Completion Implementation at SUNY. Coined the term "highly rejective." Bylined in Forbes, The Chronicle of Higher Education, Inside Higher Ed, and The Washington Post. Quoted in the NYT, The Atlantic, Vanity Fair, and the WSJ. On-screen contributor in Netflix's *Operation Varsity Blues*.

v4.0 is a clean slate, not a refactor of v3. The current site (v3.6x live, v3.69 in the client folder) is a static HTML build migrated off WordPress with no architecture planning behind it. v4.0 plans architecture first. Content ports over where useful, structure does not carry forward by default.

The site's job, in priority order:

1. Route three different audiences to the right material fast (researchers and policymakers, media and institutions, families and counselors).
2. Land paid speaking bookings. Speaking is the primary sales feature of the site.
3. Establish that the arguments are backed by actual research and data, not opinion.

---

## 1. Non-Negotiable Rules

These apply to every file, every page, every batch. Violating one is a build failure, not a style disagreement.

### Process

- **Never build with unresolved questions open.** Resolve first, then build. If something is ambiguous, stop and ask.
- **No build without an explicit go-ahead.** Do not run a build or package a release because a batch looks finished.
- **Cap any build or task list at roughly 15 items.** Longer lists produce silent errors.
- **Mechanical technical fixes can proceed without asking.** Content and voice decisions always require Akil's explicit call.
- **Never overwrite Akil's own edits.** If he has edited a file, that version is the new base.
- Increment the version number on every build.

### Writing and copy

- **No em dashes** except inside directly quoted material. Use commas, colons, or parentheticals.
- **First person throughout**, except formal bios, which are third person.
- **No eyebrow text above any headline, anywhere on the site.** No small label line, no kicker, no category tag sitting above an H1 or H2. This rule has already been used to correct three drafts. Do not reintroduce it.
- **"Writing," never "Blog." "In Public," never "Media."**
- **"Highly rejective," never "selective"** when describing elite colleges. But never make "highly rejective" the lead identifier for Akil himself.
- **"More than thirty years"** spelled out in words, always with the "more than" modifier. Numerals are fine inside stat cards ("30+").
- **Never call Twitter "X."** Always "Twitter," always the bird icon, always `twitter.com` links.
- **No emoji** unless it is clearly ironic.
- Never list FairTest as a current employer. SUNY is current. Many third-party bios are outdated on this.
- Never position Akil as a tutor or an admissions consultant.
- Never list book blurbs or supporters as confirmed. Nobody has been formally asked.
- Family details are private. Do not surface them.

### Design and behavior

- **No orange anywhere.** Mustard/gold is the pop accent. Orange was explicitly rejected for reading as urgent or caution.
- **`object-fit: cover` on every image.** No white bars, ever.
- **CTAs are contextual to a section's real content, one per hub at most.** No generic floating "book me" or "hire me" buttons. A speaking CTA only appears attached to real keynote or briefing content.
- Google Analytics goes in at build time.

### URLs and deployment


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anchordigitalco/akilbello](https://github.com/anchordigitalco/akilbello) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
