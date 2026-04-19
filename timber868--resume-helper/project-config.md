---
trigger: always_on
description: Tailor cover letters using the master cover.tex form (letter class, header, body, closer)
---


# Cover letter tailoring

When the user asks for a **tailored cover letter** for a job (or to create/update cover letters), always use the **form and structure** of the master template:

**Master template:** `data/master/cover_letter/cover.tex`

## Required form (match cover.tex)

1. **Document class and preamble**
   - `\documentclass[12pt]{letter}`
   - Same packages as in `cover.tex`: `inputenc`, `fullpage`, `hyperref`, `graphicx`, `fontawesome5`, `eso-pic`, `charter`
   - Same layout (topmargin, textheight, `gr` color for header)

2. **Personal and recipient info**
   - `\input{../../master/cover_letter/info}` from the job folder (or equivalent path from `data/jobs/<job-id>/`), then override job-specific macros:
   - `\recipient` (e.g. “Canadian Space Agency Hiring Team”)
   - `\company` if needed
   - Do not change `\myname`, `\myemail`, `\myphone`, `\mylinkedin`, `\mytitle`, `\greeting`, `\closer` unless the user asks.

3. **Header**
   - Shaded banner: `\AddToShipoutPictureBG` with `\color{gr}` and the rule as in `cover.tex`
   - Centered block: `\myname` (large, small caps), then the contact line (see below).
   - **Contact line: use icons, not plain text (mandatory).** Email, LinkedIn, and phone must be displayed **with Font Awesome icons** as in the master template, not as plain text labels like "Email:", "LinkedIn:", "Phone:". Always include the `fontawesome5` package and use:
     - `\href{mailto:\myemail}{\faEnvelope\enspace \myemail}\hfill`
     - `\href{https://linkedin.com/in/\mylinkedin}{\faLinkedinIn\enspace linkedin.com/in/\mylinkedin}\hfill`
     - `\href{tel:\myphone}{\faPhone\enspace \myphone}\hfill`
     Do **not** replace these with plain-text labels; the header must show the envelope icon, LinkedIn icon, and phone icon next to the respective contact info so the PDF matches the intended professional look.

4. **Opening block**
   - Date (e.g. current application date)
   - `\greeting\ \recipient,` in the same font size block (e.g. 14pt)

5. **Body**
   - `\setlength\parindent{24pt}` and `\fontsize{11pt}{18pt}\selectfont`
   - Tailored paragraphs for the specific job (role, company, fit, experience, closing sentence)
   - Mirror tone and structure of `data/master/cover_letter/body.txt`: opening line (application + role + dates), fit with your background, experience evidence (concrete examples), what you bring, languages/bilingual if relevant
   - Cross-check the job description (or `data/jobs/<job-id>/job_posting_extracted.json`): use wording that matches the posting’s requirements and assets without over-claiming (see “Accuracy and fit to the job posting” below).

## Output location (mandatory)

**All job-specific cover letter output must live in the same folder as the job description.**

- Job description lives in: `data/jobs/<job-id>/` (e.g. `description.txt` or `job_posting.pdf`).
- **Write only there:** Put the tailored cover letter LaTeX, body text, and compiled PDF in that same `data/jobs/<job-id>/` folder. Do **not** write job-specific cover letter content or overwrite `data/master/cover_letter/body.tex` or `data/master/cover_letter/info.tex` for a given job.
- **Output naming (mandatory) — same format as tailored resume:** Use the **same naming convention** as in `resume-helper.mdc` for the tailored resume. The only difference is the document type: **Cover** instead of **Resume**.
  - **Short posting name:** Derive it **identically** from the job folder: take the folder name (e.g. `Multi-Static Radar`, `AI and Human Factors`), replace spaces with hyphens, keep it concise (~30 chars max). Examples: `Multi-Static Radar` → `Multi-Static-Radar`; `AI and Human Factors` → `AI-and-Human-Factors`; `CSA/Space Weather` → `CSA-Space-Weather`.
  - **Filenames:** `Timothe-Roma-Cover-{Short posting name}.tex` and `Timothe-Roma-Cover-{Short posting name}.pdf`. So for the same job, the resume is `Timothe-Roma-Resume-Multi-Static-Radar.pdf` and the cover letter is `Timothe-Roma-Cover-Multi-Static-Radar.pdf`.
  - **Do not use** generic names like `cover_letter.tex` or `cover_letter.pdf`; always use the `Timothe-Roma-Cover-{Short posting name}` form.
- **Concrete outputs in the job folder:**
  - `data/jobs/<job-id>/Timothe-Roma-Cover-{Short posting name}.tex` (main letter)
  - Optionally: `data/jobs/<job-id>/cover_body.tex` or `cover_body.txt` for body-only
  - Compiled PDF: `data/jobs/<job-id>/Timothe-Roma-Cover-{Short posting name}.pdf`
- Compile from the job folder: `pdflatex "Timothe-Roma-Cover-{Short posting name}.tex"` run from `data/jobs/<job-id>/` so paths like `\input{../../master/cover_letter/info}` (or `../../../` for nested folders) resolve. The PDF is produced in that same job folder.

## Voice and style (human-written)

Cover letter body text must **sound human-written** and match the writing style of `data/master/cover_letter/body.txt`:

- **Match body.txt style:** Direct first person (“I am submitting”, “I have”, “I would bring”). Short to medium sentences. Concrete details (dates, tools, numbers) without sounding like a list. Natural transitions (“Thanks to these experiences”, “Finally,”).
- **Forbidden (enforce strictly):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Timber868) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
