---
trigger: always_on
description: >
---


# Mycelium — Living Repository Skill (Core)

When invoked, determine which mode the user needs based on their request, then follow the corresponding protocol.

For analysis, report generation, idea brainstorming, and code review, direct the user to the dedicated skills:
- `/mycelium:analyze` — start or continue an analysis
- `/mycelium:report` — generate a structured report
- `/mycelium:ideas` — brainstorm with disciplinary personas
- `/mycelium:transfer` — cross-pollinate learnings across sibling projects
- `/mycelium:review` — review code/analysis changes (PR, commit, or diff) for the kinds of mistakes that matter in scientific work; supports `grill` mode for conversational interrogation of analytical decisions

---

## Mode: `init`

**Trigger**: "set up mycelium", "initialize living repo", "restructure this repo"

**Purpose**: Scaffold a new or existing repo into a mycelium-enabled living repository.

**Steps**:
1. Check if repo already has mycelium structure (look for `.living/` directory).
2. **New repo**: Run full scaffold using `skills/core/scripts/init_repo.py`.
3. **Existing repo**: Run in restructure mode — audit current structure, propose migration plan, ask user to confirm before proceeding.
4. **Auto-install core convention packs**: Scan `network/conventions/*/CONVENTION_PACK.yaml` for packs with `core: true` and install each one using `skills/core/scripts/install_convention.py`. Currently the core packs are `robust-analysis`, `report-generator`, and `idea-generator`. These provide batteries-included practices every repo should have.
5. **Set up skillpacks directory**: Create `skillpacks/` with `.gitignore` and `README.md`. This is where external skill repos (bioSkills, scientific-agent-skills, Autonomous-Science) are cloned for use by the `skill-bridge` convention. The repos are inert reference libraries — never installed as agent skill packs. Prompt the user to clone the repos:
   ```bash
   cd skillpacks/
   git clone https://github.com/K-Dense-AI/scientific-agent-skills.git
   git clone https://github.com/GPTomics/bioSkills.git
   git clone https://github.com/arjunrajlaboratory/Autonomous-Science.git
   ```
6. Ask which **domain** conventions to install from the network (e.g., bioinformatics, image-analysis, skill-bridge). Domain packs are those without `core: true`.
7. Generate `CLAUDE.md` for the repo (from `skills/core/templates/CLAUDE.md.template`) that encodes the living repo protocol.
8. Generate `ENVIRONMENTS_INSTALLATIONS.md` at repo root.
9. Create descriptive manifests in each top-level directory (`ANALYSIS_MANIFEST.md`, `DATA_MANIFEST.md`, `ALGORITHM_MANIFEST.md`, `REFERENCE_MANIFEST.md`).
10. Initialize `.living/` with empty `decisions.md`, `learnings.md`, `conventions.md`; create `.living/log/LOG_REGISTRY.md` (session log registry — tracks work across sessions). Create `.living/outputs/knowledge-transfers/` for cross-project transfer audit trail.
11. **Bootstrap knowledge system**: If `~/.claude/knowledge/` does not exist, run `skills/core/scripts/init_knowledge.py` to set up the global progressive disclosure knowledge system. The script also appends the Global Knowledge Domains routing table to every `~/.claude/projects/*/memory/MEMORY.md` (idempotent — skips files where the header is already present). Generate `.living/INDEX.md` for the newly scaffolded project using `skills/core/scripts/generate_index.py --summary-heuristic`.
12. Create `todo/` directory with `TODO_REGISTRY.md` (registry table) and `TODO_ITEM_TEMPLATE.md` (template for individual items). Copy these from the mycelium `todo/` directory.
13. After completion: run `skills/core/scripts/validate_structure.py` to confirm everything is correct.

**References to consult**:
- `skills/core/references/folder-structure.md` — canonical target structure
- `skills/core/references/environment-setup.md` — environment setup conventions

---

## Mode: `ingest`

**Trigger**: "ingest dataset", "add data", "import data"

**Purpose**: Pull a new dataset into the analytical framework.

**Steps**:
1. Consult `skills/core/references/data-ingest-conventions.md`.
2. Determine data type, source, and format.
3. If a domain convention is active, check its conventions for domain-specific validation.
4. Place raw data in `data/raw/[dataset-name]/`.
5. Generate metadata in `data/metadata/[dataset-name]/` using templates: `skills/core/templates/schema.yaml` for column definitions, `skills/core/templates/provenance.md` for source documentation, and `skills/core/templates/summary_stats.md` for statistical overview.
6. Update `data/DATA_MANIFEST.md` with new entry (use `skills/core/templates/dataset-manifest-entry.yaml`).
7. Log any decisions about data cleaning or exclusion to `.living/decisions.md`.
8. Run the post-action hook protocol (see below).

---

## Mode: `install-convention`

**Trigger**: "install convention", "add bioinformatics conventions", "install convention pack"

**Purpose**: Install a convention pack from the mycelium network into the current repo.

**Context**: Core packs (`robust-analysis`, `report-generator`, `idea-generator`) are auto-installed during `init`. This mode is primarily for adding domain packs after initialization, but can also be used to manually install or reinstall any pack.

**Steps**:
1. Consult `skills/core/references/marketplace-guide.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arjunrajlaboratory/mycelium](https://github.com/arjunrajlaboratory/mycelium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
