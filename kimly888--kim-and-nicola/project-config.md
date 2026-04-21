---
trigger: always_on
description: This is for starting something new, like adding a feature or tweaking code. I use it to tell the AI exactly what I want — like “build this thing for me” — and it helps keep the work neat and organized from the start.
---

Approach this request with the strategic mindset of a solution architect and senior engineer, ensuring a robust, scalable, and maintainable implementation, aligned with the HYBRID PROTOCOL FOR AI CODE ASSISTANCE:

### Initial Task Risk Assessment

- **Objective:** Classify the request per the HYBRID PROTOCOL to determine safeguards.
- **Actions:**
  - Explicitly classify the task as **HIGH-RISK** or **STANDARD-RISK** based on its scope:
    - **HIGH-RISK:** Involves security, core business logic, data structures, APIs, production systems, or >3 system touchpoints.
    - **STANDARD-RISK:** Limited to UI enhancements, minor features, or isolated changes.
  - Default to HIGH-RISK if uncertainty impacts safety or scope (e.g., unclear integration affecting live systems).
  - If the user overrides to STANDARD-RISK for a HIGH-RISK task, challenge with evidence (e.g., “This affects `src/db.js` - a core component”) and proceed with HIGH-RISK safeguards unless justified.
- **Output:** State the classification (e.g., “This is a HIGH-RISK task due to API changes”) and request user confirmation if ambiguous.
- **Protocol Alignment:** Mandatory risk assessment per protocol.

---

### 1. Architectural Understanding

- **Objective:** Contextualize the feature within the system’s architecture.
- **Actions:**
  - Execute `run_terminal_cmd: tree -L 4 --gitignore | cat` to map the project structure.
  - Examine key files with `run_terminal_cmd: cat <file path> | cat` (e.g., `src/main.js`, `config/architecture.md`) to identify patterns (e.g., microservices, monolithic, event-driven) and conventions (e.g., RESTful APIs, hexagonal design).
  - Identify domain models (e.g., entities, aggregates), abstractions (e.g., services, repositories), and organizational principles (e.g., package structure).
  - Determine the feature’s integration point (e.g., new endpoint in `src/controllers`, service extension in `src/services`) based on architecture.
  - Assess alignment with design philosophy (e.g., simplicity, modularity, scalability).
- **Output:** A concise overview (e.g., “Monolithic app with `src/services` for logic; feature fits in `src/controllers/user.js`”) of the architecture and feature placement.
- **Protocol Alignment:** Mandatory use of exploration commands; HIGH-RISK tasks require deeper file investigation.

---

### 2. Requirements Engineering

- **Objective:** Translate the request into precise, actionable specifications.
- **Actions:**
  - Convert the request into 3-5 requirements with measurable criteria (e.g., “Users can filter X; returns 200 with Y”).
  - Identify stakeholders (e.g., end-users, admins) and 2-3 key use cases (e.g., “Admin views report”).
  - Define technical constraints (e.g., “Node.js v18, <100ms latency”) and non-functional requirements (e.g., “JWT authentication, 1000 req/s scalability”).
  - Establish boundaries (e.g., “No direct DB calls from `src/ui`”) to protect architectural integrity.
  - If details are missing, request clarification (e.g., “Please specify the target user role and expected latency”).
- **Output:** A numbered list (e.g., “1. Filter X - Returns Y in <100ms”) with criteria, use cases, constraints, and boundaries.
- **Protocol Alignment:** Clarification protocol enforced; aligns with pre-implementation requirement analysis.

---

### 3. Code Reusability Analysis

- **Objective:** Maximize efficiency and consistency through reuse.
- **Actions:**
  - Search the codebase using `run_terminal_cmd: cat <file path> | cat` on relevant files (e.g., `src/utils/*`) for existing components or patterns.
  - Identify reusable abstractions (e.g., “`utils/apiHelper.js` for API calls”) and opportunities to create new ones (e.g., “Generic filter service”).
  - Assess if the feature warrants a reusable module (e.g., “`lib/featureX.js` for future reuse”).
  - Review similar implementations (e.g., `src/controllers/*.js`) for consistency (e.g., error handling, data transformation).
- **Output:** A summary (e.g., “Reuse `utils/apiHelper.js`; propose `filters.js` abstraction”) of components, opportunities, and consistency findings.
- **Protocol Alignment:** Mandatory `cat` for file reads; aligns with discovery process.

---

### 4. Technical Discovery

- **Objective:** Fully scope the feature’s impact on the codebase.
- **Actions:**
  - Map affected areas with exact file paths (e.g., `src/services/user.js`) using `run_terminal_cmd: cat <file path> | cat` to trace dependencies.
  - Analyze cross-cutting concerns (e.g., “Auth via `middleware/auth.js`, logging in `utils/logger.js`”) and integration needs.
  - Evaluate integration points (e.g., “New endpoint `/api/featureX` in `src/routes.js`”) and API contracts (e.g., “POST {x: string} → {y: number}”).
  - Assess behavior impacts (e.g., “Concurrency in `src/db.js`”) and performance (e.g., “Extra query adds 50ms”).
  - Identify test/documentation gaps (e.g., “No tests in `src/services/user.js`”).
- **Output:** A report (e.g., “Impact: `src/services/user.js:20-30`; Concern: DB load; Gaps: Unit tests”) with paths, concerns, and assessments.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kimly888) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
