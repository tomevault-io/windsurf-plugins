---
trigger: always_on
description: Generates ATS-optimized resume and cover letter in LaTeX from a job description URL or pasted text.
---


# ATS-Optimized Resume & Cover Letter Generator

Generate a tailored LaTeX resume and cover letter from a job description, optimized for ATS screening while sounding authentically human.

> **Portability:** This is a tool-agnostic skill. It needs four generic capabilities — fetch a URL, read files, write files, and run a shell — plus the ability to ask the user a question. The `allowed-tools` frontmatter lists those capabilities using Claude Code's tool names for convenience; any agent (Codex, etc.) can run the skill by mapping them to its own equivalents. Wherever this file says "your skill directory", it means the folder this `SKILL.md` lives in.

> **Setup before first use:** Fill in `assets/profile.md` with your own background (work history, skills, education, awards). The shipped `profile.md` is an empty template with section scaffolding only. Optionally edit `assets/resume-template.tex` to replace the placeholder contact block with your name and links. `assets/lessons.md` starts empty and grows as you give feedback (Phase 8).

---

## Phase 1: Input Acquisition

Determine how to obtain the job description:

- If `$ARGUMENTS` is empty or missing, ask the user to provide a job description (URL or pasted text).
- If `$ARGUMENTS` starts with `http://` or `https://`, fetch the page content with your web-fetch capability. If fetching fails (login wall, paywall, 403, etc.), inform the user and ask them to paste the JD text directly.
- Otherwise, treat `$ARGUMENTS` as the raw job description text.

---

## Phase 2: JD Analysis

Extract the following from the job description:

- **Company name**
- **Role title**
- **Required skills** (must-haves)
- **Preferred skills** (nice-to-haves)
- **Key responsibilities**
- **Culture keywords** (values, team style, mission language)
- **Seniority level** (inferred from title, years of experience, scope)

Present a short summary to the user and ask them to confirm or correct before proceeding:

> **Company:** X | **Role:** Y | **Level:** Z
> **Must-have skills:** ...
> **Nice-to-have skills:** ...
> **Key themes:** ...
>
> Does this look right? Any adjustments?

**Spotting hidden optional skills:** Some JDs blur the line between required and preferred. When a sentence like "Experience or interest within the above-mentioned areas is a good foundation" follows a skill list, those skills are effectively **optional**, not hard requirements. Read the prose, not just the "Required / Preferred" headings, and classify accordingly in your summary.

---

## Phase 3: Profile & Template Loading

Read the following files from the `assets/` folder in your skill directory:

1. **`profile.md`** — Personal information and experience inventory. If this file is missing or mostly empty, STOP and tell the user:
   > Your profile is not set up yet. Please fill in `assets/profile.md` with your information first.
2. **`resume-template.tex`** — LaTeX resume template.
3. **`cover-letter-template.tex`** — LaTeX cover letter template.
4. **`lessons.md`** — Past feedback and learned preferences. If missing or empty, skip silently — this is expected for first-time use.

When `lessons.md` has entries, treat every rule in it as a hard constraint during generation.

**Note on multilingual profile content:** `profile.md` may be written in a language other than English (or a mix). Read and fully understand all of it — do not skip or skim sections because they are not in English. When generating English output, translate accurately and preserve all details, including metrics, project names, technical specifics, and competition results.

---

## Phase 3.5: Pre-Generation Preview (MANDATORY)

Before writing any file, present the planned approach to the user and wait for explicit confirmation.

The preview must cover:

**Resume strategy:**
- Positioning angle of the Summary (which identity/background to lead with)
- Which experience is the core (most relevant), which to down-weight or omit
- How the Skills section is grouped and which skills are emphasized
- Whether Certifications are kept and how they're presented

**Cover letter strategy:**
- Opening positioning (identity + career-arc sentence if the path is non-linear + strongest match + why this company)
- The 2–3 core JD requirements the body covers, the experience mapped to each, and its impact (what you did → what changed)
- Personal-connection detail (if any, capped at 1–2 sentences, noting which paragraph it goes into)
- Closing handling (language ability, availability, etc.)

Present this as a concise bullet-point outline (in the user's working language), then ask:

> Does this approach look right? Once you confirm, I'll start generating the files.

**List every work-experience entry from `profile.md` and state how each is handled (kept / down-weighted / omitted) with a reason.** Do not silently drop a role, even a low-relevance one — name it and explain, and let the user decide whether to include it.

**Do not write any `.tex` file until the user confirms.**

---

## Phase 4: Resume Generation

### Content Strategy

- Review ALL experience entries in `profile.md`. Select and prioritize bullets that best match the JD's required and preferred skills.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1carusalwayswa/apply](https://github.com/1carusalwayswa/apply) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
