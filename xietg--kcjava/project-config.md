---
trigger: always_on
description: This file is the **single place** for product intent, repo facts, and **your expectations**. Keep it short and current; link to GitHub issues or docs for long specs.
---

# Claude.md — project ground truth

This file is the **single place** for product intent, repo facts, and **your expectations**. Keep it short and current; link to GitHub issues or docs for long specs.

**How assistants should use it:** treat this as authoritative for goals, workflow, and conventions; **verify implementation in code** when changing behavior. If the codebase lags this document, the doc describes **target product** — call out gaps in issues rather than silently shrinking scope.

---

## 1. Project scope

### 1.1 Product (target)

Web application to **collect vendor questionnaire responses**, **score** answers, and **rank** vendors within a **market segment**. Questionnaires are delivered as Excel workbooks that **always** use two fixed sheet names for machine-readable question definitions — see **§1.4**.

**Business capabilities (product backlog — not all may be implemented yet):**

1. Create a new market segment (Leadership Compass / **LC**).
2. Attach a questionnaire to a market segment.
3. Create a new questionnaire.
4. Import an existing questionnaire from an **XLSX** file.
5. Edit **technical** questions for a questionnaire.
6. Update **standard** questions for **new** questionnaires without changing historical questionnaires.
7. Download a questionnaire as **XLSX** for distribution to vendors.
8. Import a vendor-filled **XLSX** as a submission.
9. Define **scoring categories** per market segment.
10. Score each question **0–1** in **zero or more** categories.
11. Compute **overall** scores per category.
12. Build **charts** for overall results.

**Channels:** a **public REST API** and a **web frontend**.

**Users:** multiple roles are expected; **role model is not final yet** (evolve with issues, keep API extensible).

**Integrations (planned / future):** e.g. **Salesforce** (vendor master data), **enterprise IDP** (identity). Today the app uses **local JWT + DB users** unless/until IDP is wired.

**Non-functional:**

- **Robust** to invalid or malformed input files (clear errors, no silent corruption).
- **Scale:** on the order of **~20 concurrent users** for the MVP trajectory; architecture should not block **SaaS on Azure** later.
- **UX principle:** as **simple** and **reliable** as possible for operators and vendors.

### 1.2 Delivery governance

- **GitHub Issues** are the **backlog source of truth**: all substantive work should be **tracked**, **prioritized**, then executed.
- Prefer **one issue per deliverable** (or a clear parent/child breakdown) so history and scope stay auditable.

### 1.3 Current codebase vs product vision

- This repo is a **Spring Boot backend** (MVP port) with **Postgres**, **JWT auth**, **XLSX** via Apache POI, and **OpenAPI/Swagger**.
- **Frontend** exists under `frontend/` (see repo map). Features in §1.1 items **9–12** (scoring model, aggregates, charts) may be **partial or future** — implement per prioritized issues; do not assume they already exist.

### 1.4 Questionnaire XLSX — sheet names and layout (canonical)

**Fixed tab names (non-negotiable for parsers and template generation):**

- **Standard questions (shared across market segments):** sheet name **`4. Standard Questions`**
- **Technical questions (segment-specific):** sheet name **`5. Technical Questions`**

Code that imports or exports the **question definition** must target **only** these two sheets by **exact name**. Any other sheets in the same workbook are **out of scope** for question-definition I/O unless explicitly specified in a future issue.

**Typical full workbook shape:** a complete vendor-facing file may include **additional human-facing sheets** (e.g. introduction, process, market segment context, contact). Those sheets may contain **images and prose**; the backend must **not** rely on them for structured question data. Parsing logic should **ignore** everything outside the two canonical tabs above when reading or validating **question rows**.

**Row semantics (both tabs, aligned with `ExcelParser`):** rows mix **section headers (“categories”)** and **questions**. Implementations should follow the same rules as today’s parser:

- **Standard tab (`4. Standard Questions`):** a **category** row has **empty** column A, **category title** in column B, and **empty** C and D. A **question** row has **question index** in A (e.g. `1)`), **wording** in B, **type** in C, **help text** in D. **Sub-questions** reuse the parent index from A and use **empty** A with B/C/D filled; the parser synthesizes sub-indices (`a)`, `b)`, …).
- **Technical tab (`5. Technical Questions`):** a **category** row has **category title** in column A and **empty** B, C, D. **Question** and **sub-question** rows follow the same four-column pattern as on the standard tab (index / text / type / help).

**Question `type` values** are stored as strings and must stay consistent with the domain (see DB comments and existing data — e.g. text, numeric, percentage, boolean, comment, choices). Invalid or unknown types should surface as **clear parse or validation errors**, not silent coercion.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xieTG) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
