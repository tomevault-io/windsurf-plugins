---
trigger: always_on
description: This document defines the workflows for generating tailored resumes and polishing timeline events.
---

# GitHub Copilot Project Rules & Resume Generation Workflow

This document defines the workflows for generating tailored resumes and polishing timeline events.

## Agent Selection

Based on the user's input, determine which agent to activate:

1.  **Resume Generation Agent**: Triggered when the user provides a **Job Description (JD)**.
2.  **Timeline Polishing Agent**: Triggered when the user provides raw **Work Experience** or **Project** descriptions for polishing.
3.  **Interview Preparation Agent**: Triggered when the user provides a **Resume**, **JD Analysis**, and **Company Business Analysis** to generate an interview guide.

---

## Agent 1: Resume Generation Agent

**Goal**: Create a highly targeted, professional resume in YAML format that matches a specific Job Description.

### Workflow Steps

#### 1. Initialization

- **Action**: Create a temporary directory to store intermediate generation artifacts.
- **Path**: `resumes/temp/{Timestamp}/` (e.g., `resumes/temp/1704067200/`).

### 2. Job Analysis

- **Action**: Analyze the provided JD to extract key requirements, skills, role details, and **detect the language**.
- **Tool**: Use the prompt defined in `resumes/job-analysis-prompt.md`.
- **Input**: The user-provided JD.
- **Output**: Save the analysis result to `resumes/temp/{Timestamp}/job-analysis.yml`.

### 3. Company & Business Analysis

- **Action**: Analyze the JD to identify the company, search for business context, and infer product/business unit details.
- **Tool**: Use the prompt defined in `resumes/company-business-analysis-prompt.md`.
- **Input**: The user-provided JD.
- **Output**: Save the analysis result to `resumes/temp/{Timestamp}/company-business-analysis.yml`.

### 4. Context Gathering & Timeline Matching

- **Action**: Gather candidate information and select relevant experience.
- **Sources**:
  - **Basics**: Read `profiles/basics.yml`.
  - **Education**: Read `profiles/education.yml`.
  - **Certificates**: Read `profiles/certificates.yml`.
  - **Timelines**: Read all files in `timelines/gem/`.
- **Matching**: Based on the **Job Analysis** and **Company Business Analysis** results, select the most relevant Work Experience and Project files from `timelines/gem/`. Filter out irrelevant experiences if necessary, or prioritize relevant ones.

### 5. Section Generation

Generate each section of the resume using the specific prompt files. Pass the **Job Analysis** (including `language`), **Company Business Analysis**, and **Matched Context** (Profile, Education, Selected Timelines) to each prompt. Save the output of each section to the temporary directory.

**Order of Generation:**

1.  **Projects**: Generate first to establish technical depth.
2.  **Work Experience**: Generate second, referencing project highlights if applicable.
3.  **Skills**: Generate third, extracting keywords from both Projects and Work Experience.
4.  **Personal Summary**: Generate LAST. **Crucial**: You MUST pass the _generated_ content of Projects, Work Experience, and Skills as context to this prompt to ensure the summary accurately reflects the tailored resume.

- **Projects**:
  - **Prompt**: `resumes/section-projects-prompt.md`
  - **Output**: `resumes/temp/{Timestamp}/section-projects.yml`
- **Work Experience**:
  - **Prompt**: `resumes/section-work-prompt.md`
  - **Output**: `resumes/temp/{Timestamp}/section-work.yml`
- **Skills**:
  - **Prompt**: `resumes/section-skills-prompt.md`
  - **Output**: `resumes/temp/{Timestamp}/section-skills.yml`
- **Personal Summary**:
  - **Prompt**: `resumes/section-personal-summary-prompt.md`
  - **Input**: Pass `section-projects.yml`, `section-work.yml`, and `section-skills.yml` (generated above) in addition to the standard context.
  - **Output**: `resumes/temp/{Timestamp}/section-personal-summary.yml`

### 6. Resume Assembly

- **Action**: Assemble the final resume using the intermediate files from `resumes/temp/{Timestamp}/`.
- **Sources**:
  - `profiles/basics.yml` (Basics)
  - `profiles/education.yml`
  - `profiles/certificates.yml`
  - `resumes/temp/{Timestamp}/section-personal-summary.yml`
  - `resumes/temp/{Timestamp}/section-skills.yml`
  - `resumes/temp/{Timestamp}/section-work.yml`
  - `resumes/temp/{Timestamp}/section-projects.yml`
- **Constraint**: DO NOT read or use any `*.example.yml` files.
- **Template**: Follow the structure of `resumes/resume.example.yml` (Structure ONLY, do not read content).
- **Section Order**: The `content` section MUST strictly follow the order defined in `layouts.sections.order` (basics -> education -> work -> skills -> certificates -> projects).
- **Structure**:

  ```yaml
  locale:
    # Use `yamlresume languages list` to get the list of supported languages
    # Set this based on the user's request or the detected language from Job Analysis
    # Supported: en, zh-hans, zh-hant-hk, zh-hant-tw, es, fr, no
    language: en

  layouts:
    - engine: latex
      sections:
        aliases:
          basics: 'Basics'
          education: 'Education'
          work: 'Work Experience'
          projects: 'Projects'
          certificates: 'Certificates'
          skills: 'Skills'
          languages: 'Languages'
          interests: 'Interests'
        order:
          - basics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhiweio/resume-as-code](https://github.com/zhiweio/resume-as-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
