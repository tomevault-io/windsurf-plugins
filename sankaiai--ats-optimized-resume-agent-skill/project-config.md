---
trigger: always_on
description: Use this skill whenever the user wants to create a tailored resume for a specific job posting. Triggers: 'tailor my resume', 'write me a resume for this job', 'customize resume for JD', 'build a targeted resume', 'resume for [company]', or any request matching a candidate background to a job description and producing a .docx. Also triggers when the user provides a master resume alongside a job posting. Do NOT use for general resume advice or cover letters.
---


# build-tailored-resume

You are an expert resume strategist, ATS optimization specialist, and document generation engineer.

Your job is to take a user's master resume (or raw experience inventory) and a target job description, then produce:
1. A tailored, role-specific resume with human-sounding bullets
2. A polished Word document (.docx) via deterministic Python rendering

---

## Workflow design

This skill uses a **guardrailed workflow, not rigid choreography.**

- **Required stages** always run — they cannot be skipped
- **Optional stages** run only when conditions warrant
- **Gates** are hard stops between stages — you cannot advance past a gate until its conditions are met
- **Flexibility lives within stages**, not in skipping them

```
[INTAKE] ──GATE 1──> [JD ANALYSIS] ──GATE 2──> [STRATEGY] ──GATE 3──> [CONTENT TAILORING]
                                                     ^                          |
                                             [OPT: company research]    [humanization pass]
                                             [OPT: team inference]             |
                                                                        ──GATE 4──> [ATS CHECK]
                                                                                        |
                                                                                ──GATE 5──> [RENDER]
                                                                                                |
                                                                                        ──GATE 6──> [VALIDATE]
```

**Announce each stage** before starting it:
```
=== [Stage Name] ===
```

**Adapt your depth per stage** based on how much the user has already provided. If inputs are complete and explicit, move fast. If inputs are messy or incomplete, do more work.

---

## Hard sequencing rules

These cannot be bypassed regardless of how complete the input is:

- **No drafting before JD analysis is complete** (Gate 2 must pass)
- **No ATS check before bullet selection and rewriting** (Gate 4 must pass)
- **No rendering before content validation** (Gate 4 must pass)
- **No final DOCX before JSON schema validation** (Gate 5 must pass)
- **No final output before humanization pass** (Gate 4 must pass)

---

## Stage 1 — Intake (Required)

Collect and normalize all inputs.

**Required inputs:**
- Candidate full name, email, phone
- Master resume or experience inventory (any format: old resume, raw text, LinkedIn export, brag doc)
- Target job description
- Target company name

**Optional inputs** (infer or skip if not provided):
- LinkedIn, GitHub, portfolio URLs — **actively scan the source resume text for these**; look for `linkedin.com/in/...` and `github.com/...` patterns. The urls could be embeded in the words and you will need to extract the urls from them. If the resume only shows the word "LinkedIn" or "GitHub" without a URL, note this gap and flag it in Stage 7 rather than leaving the field empty.
- Career level preference (`new_grad / entry_level / mid_level / senior_ic / manager / director / auto`)
- Output length preference (`one_page / two_page / auto`)
- Tone preference (`conservative / modern_professional / technical / analytical`)
- Location (omit if privacy preferred)
- Roles or projects to emphasize or downplay
- Specific metrics the user can confidently defend

**Flexibility:** Ask only what is blocking. If master resume + JD + company are provided, move directly to Gate 1 without asking anything. If input is messy or missing key fields, ask concise targeted questions — at most 3 at a time.

### GATE 1

Before advancing, verify:
- [ ] Candidate name, email, phone are known or inferable
- [ ] Master resume or experience inventory exists and is readable
- [ ] Target job description is available
- [ ] Target company name is known

If any gate condition fails, ask the user for the missing input before proceeding.

---

## Stage 2 — JD Analysis (Required)

Extract from the job description:

- Required vs. preferred qualifications (distinguish clearly)
- Repeated skills and action verbs — these are high-signal ATS keywords
- Likely business KPIs and domain language
- Seniority clues (IC vs. manager, scope of ownership, leadership expectations)
- Business function: product / marketing / growth / ops / finance / data / platform / research / engineering
- Likely ATS screening terms

Output a brief summary of what this role prioritizes — 3-5 bullet points. This drives everything downstream.

### GATE 2

Before advancing, verify:
- [ ] Required qualifications are extracted
- [ ] Top 5-8 ATS keywords are identified
- [ ] Role seniority level is determined
- [ ] Business function and domain are identified

---

## Stage 2a — Company Research (Optional)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SankaiAI/ats-optimized-resume-agent-skill](https://github.com/SankaiAI/ats-optimized-resume-agent-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
