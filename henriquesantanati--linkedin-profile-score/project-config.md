---
trigger: always_on
description: Audit and score a LinkedIn profile from its native CSV data export. Analyzes profile sections for recruiter visibility, keyword optimization, and credibility. Use when the user provides a folder of CSV files from a LinkedIn data export and wants to optimize their profile for job searching. Triggers on: audit my LinkedIn, review my profile, LinkedIn optimization, profile score, score my profile, LinkedIn audit, optimize my LinkedIn, LinkedIn CSV. Do NOT use for writing LinkedIn posts, managing co
---


# LinkedIn Profile Score

Analyze a LinkedIn profile from its native CSV data export and produce a comprehensive score with actionable recommendations to improve recruiter visibility and hiring chances.

## Prerequisites

This skill requires the user's LinkedIn data exported as CSV files (the default format LinkedIn provides).

**How to get the data:**
1. Go to LinkedIn → Settings → Data Privacy → Get a copy of your data
2. Select "Download larger data archive" and request the archive
3. Wait for the email (usually 24-72h)
4. Download and unzip the ZIP file
5. Provide the path to the unzipped folder

**Expected CSV files** (read all that exist in the export folder):

| File | Content |
|------|---------|
| `Profile.csv` | Name, headline, location, summary, industry |
| `Positions.csv` | Work history (title, company, description, dates) |
| `Skills.csv` | Listed skills |
| `Connections.csv` | Network connections |
| `Shares.csv` or `Posts.csv` | Posts and activity |
| `Company Follows.csv` | Companies and organizations followed |
| `Member_Follows.csv` | People followed |
| `Languages.csv` | Profile language proficiency |
| `Certifications.csv` | Licenses and certifications |
| `Education.csv` | Educational background |
| `Email Addresses.csv` | Contact email(s) |

Note: LinkedIn occasionally changes file names between export versions. If a file is not found by exact name, look for partial matches (e.g., a file containing "position" or "experience" in its name). Adapt to whatever CSV files are present.

## Workflow

### Step 1: Locate Input

Ask the user for the path to their unzipped LinkedIn data export folder. List the CSV files found and confirm which ones are available before proceeding.

**Privacy note:** Inform the user that their export contains personal data (name, email, work history, connections). This data is processed locally by the LLM and is not stored or shared, but the user should be aware of what they are providing.

### Step 2: Read Profile Data

Read all available CSV files. Parse them as standard CSV (comma-delimited, with headers in the first row). Track which files are present and which are missing. Missing files mean those sections receive no score (not zero, just excluded from the weighted calculation and weights redistributed proportionally).

### Step 3: Automated Analysis

Analyze 15 sections using the criteria in `references/scoring_rubric.md`. For each section:
- Assign a score from 1 to 10 (use rubric 1-5 anchors as guidance, full 1-10 range for granularity)
- Identify specific issues found
- Write one concrete recommendation (what to change and why)

If the user specified a target role, incorporate keyword gap analysis directly into each section's recommendation. When suggesting a rewritten headline, About section, or experience bullet, include the missing keywords naturally. Do not create a separate keywords section — the fixes should already contain the right keywords.

The 15 sections and their weights:

| Section | Weight | Source File |
|---------|--------|-------------|
| Headline | 15% | Profile.csv |
| Skills | 13% | Skills.csv |
| Experience Descriptions | 12% | Positions.csv |
| About | 10% | Profile.csv |
| Experience Titles | 8% | Positions.csv |
| Connections | 8% | Connections.csv |
| Experience Duration | 5% | Positions.csv |
| Activity | 5% | Shares.csv / Posts.csv |
| Interests | 4% | Company Follows.csv + Member_Follows.csv |
| Profile Language | 4% | Languages.csv |
| Location | 4% | Profile.csv |
| Contact Info | 4% | Profile.csv + Email Addresses.csv |
| Certifications | 3% | Certifications.csv |
| Education | 3% | Education.csv |
| Name | 2% | Profile.csv |

### Step 4: Manual Checklist

Present 7 items that cannot be verified from the data export. These are things the user needs to check themselves. For each item, explain:
1. Where to find it (exact navigation path in LinkedIn)
2. What to look for
3. What the recommendation is if it needs fixing

Read `references/manual_checklist.md` for the full checklist with context. Do NOT present these as yes/no results — you cannot verify them. Present them as a clear guide the user should follow after fixing the automated issues.

### Step 5: Calculate Score and Generate Report

**Score formula:**
- Score each section on a 1-10 scale using the criteria in scoring_rubric.md as guidance (rubric defines 1-5 anchors; use the full 1-10 range for finer granularity)
- Final score = weighted average of all section scores, expressed as a percentage (0-100)
- Formula: sum(section_score × section_weight_as_decimal) / 10 × 100. Weights are decimals: 15% = 0.15. Example: if weighted sum is 6.7, score = 67/100
- If data is missing for a section, exclude it and redistribute its weight proportionally among remaining sections

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [henriquesantanati/linkedin-profile-score](https://github.com/henriquesantanati/linkedin-profile-score) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
