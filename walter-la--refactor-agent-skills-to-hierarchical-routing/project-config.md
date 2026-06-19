---
trigger: always_on
description: name: refactor-agent-skills-to-hierarchical-routing
---

---
name: refactor-agent-skills-to-hierarchical-routing
description: |
  Refactor flattened AI Agent Skills into an enterprise-grade contract of “Hierarchical Toolkit Routers + a Global Root Router”:
  - Global routing (Root Router): cross-domain intent dispatching, cross-toolkit pipeline orchestration, and state propagation
  - Type validation (a machine-verifiable subset of JSON Schema)
  - DAG dependency / cycle protection (including an interactive de-cycling contract + unattended fallback)
  - Parallel scheduling (governing both data dependencies + side effects / resource mutual exclusion)
  - Security redaction (declarative + enforced by external interceptors in the Host Environment)
  - Coverage conservation (merge/split allowed, but nothing may be dropped; verifiable)
  - Idempotency / retries / compensation (prevent retry-induced side-effect disasters)
  - Observability (mandatory inheritance of trace_id / span_id)
  Produce Step0–5 full YAML contracts and a verifiable lint.
schema_version: "2.2"
---

# Skill: Refactor to Hierarchical Router

## 0) Goals (MUST)

### 0.1 Toolkit Goals
- **Cohesion over Fragmentation**: If a cohesive domain (e.g., skills sharing a common prefix or purpose) contains ≤ 30 skills, group them into a SINGLE toolkit. Do NOT artificially split them just to create multiple toolkits.
- If the entire skill inventory is large and diverse, produce 2–5 `<domain>-toolkit` toolkits.
- Each toolkit MUST have ≤ 30 sub-skills; `misc-toolkit` ≤ 20.
- Each `<domain>-toolkit` MUST contain a `SKILL.md` file that acts as the routing entry point for that domain. The `description` field in its YAML is CRITICAL and must accurately reflect the intent of all its sub-skills to ensure the Host Agent routes to it correctly. Furthermore, its markdown body MUST contain a `sub_skills` index registry so the Agent knows exactly where to route.

### 0.2 Output Format & Verifiability
- Step0–5 outputs must each be **a single parseable YAML**
- Markdown may only be embedded via YAML block scalars (`|` or `>`)
- Must be parseable by `yaml.safe_load` (Step5 MUST verify)

### 0.3 Correctness & Safety
- Coverage conservation: must not omit any input skill (merge/split allowed but must be explicitly declared)
- Concurrency safety: parallelism requires BOTH “no data dependency” + “no side-effect/resource mutex conflict”
- Security redaction: logs/handoff/final_outputs must redact sensitive values
- Cycle protection: if a cycle exists, MUST output a `cycle_resolution` contract; and provide an **unattended fallback**
- Idempotency & retries: any skill that may produce side effects MUST declare `idempotent`; non-idempotent skills MUST NOT be retried unless compensation is provided

---

## 1) Hard Rules (MUST)

### 1.1 File Naming & Bundle Preservation (MUST)
- The toolkit root directory (`<domain>-toolkit/`) MUST contain its router file exactly named `SKILL.md`. It MUST NEVER be named `<domain>-toolkit.md`.
- You do NOT need an artificial `sub_skills/` folder. Sub-skills should be placed directly inside the toolkit directory as subfolders (e.g., `<domain>-toolkit/old_folder_name/`).
- **Host Loader Protection (Sub-skills only)**: Under the toolkit subfolders, the entry point `.md` for **sub-skills** MUST NOT be named `SKILL.md` or any reserved name (`ROUTER.md`, `INDEX.yaml`). This prevents naive Host File Loaders from using `glob("**/*.md")` and confusing sub-skills with master routers. Rename the entry point to match the folder name (e.g., `my_skill/SKILL.md` -> `my_skill/my_skill.md`).
- **Bundle Completeness**: If the original skill was a folder, the ENTIRE folder (including all scripts, schemas, and helper files) MUST be preserved and moved together.

### 1.2 File Encoding & Artifact Cleanup (MUST)
- **UTF-8 Encoding**: When reading or modifying files, the Agent MUST preserve file encoding. All file modifications MUST explicitly use UTF-8 encoding (without BOM) to prevent non-ASCII characters (like Chinese text) from becoming garbled.
- **Cleanup**: After refactoring, the Agent MUST clean up any temporary artifacts generated during the process (e.g., delete `step0-5-contracts.yaml` left in the toolkit directory, and delete any temporary migration scripts like `refactor_*.ps1` left in the `skills` directory).

### 1.3 Shared Resources & Path Rewriting (MUST)
- During Step 0 (Cleansing), the Agent MUST statically analyze the skill documents to discover any relative path references to external/shared folders (e.g., `../shared_utils/`).
- If shared resources exist, they MUST be migrated into the toolkit (or kept global if cross-toolkit), BUT they MUST keep their original folder name (e.g., if the folder was `shared_utils`, it stays `shared_utils` and is NOT renamed to `shared`).
- The Agent MUST safely rewrite any broken relative paths inside the migrated `.md` or code files to correctly point to the moved shared resources.

### 1.4 Cleansing (Minimum Viable)
Each skill MUST have at least:
- `name`
- `description` (must include: action + object + deliverable)
- `required_inputs[]`, `outputs[]` (if missing, fill them and mark `confidence: low`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [walter-la/refactor-agent-skills-to-hierarchical-routing](https://github.com/walter-la/refactor-agent-skills-to-hierarchical-routing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
