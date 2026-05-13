---
trigger: always_on
description: - **Project Structure**: This project enforces a maximum depth of 2 levels for nested project directories.
---

# GEMINI Dev Guidelines

## Project Capabilities & Constraints

- **Project Structure**: This project enforces a maximum depth of 2 levels for nested project directories.
- **Task & Dependency Management**: We are using the `beads` CLI to manage tasks and dependencies.
- **Product Requirements**: The `README.md` file serves exactly as the Product Requirements Document (PRD).
- **Design System**: Elements and components must follow the Design System located at `docs/design`. This is the "document" for all designs to follow, reducing the gap between the PRD and the real product.
- **Documentation Ground-Truth**: All architectural, pedagogical, and system design documentation (other than PRD) is centralized exclusively in the `./docs` directory. AI Agents must reference this directory for project context and component logic.

### Documentation Directory Mapping (`./docs/`)
To ensure knowledge is centralized and updated correctly, ALL Agents must adhere to the following directory structure inside `docs/` for reading context and creating new files:

- **`docs/architecture/`**: Detailed architectural blueprints, component logic, game mechanics flow, and engineering decisions. Agents should read this to understand *how* the app works underneath and update it when creating new core components.
  - `game_journey_logic.md`: Details the component logic, filtering conditions, and rendering constraints for the pedagogical map (Game Journey).
  - `runner-game-engine.md`: Details the physics loop, entity states, and obstacle configuration for the Endless Runner game wrappers.
- **`docs/design/`**: The Wordlist Quiz Design System. Contains foundational UI tokens, CSS layouts, and HTML showcases. Agents MUST update this directory BEFORE modifying any UI components in `src/`.
  - `README.md`: The core rulebook and ID registry (124+ nodes) mapping the UI components to React.
- **`docs/requirements/`**: Source requirements, PRDs, and external datasets. Read-only context for data ingestion.
  - `study-materials/grade3/esl/learning-objectives.md`: Baseline curriculum mapping and objectives for Grade 3 English acquisition.
- **`docs/api/`**: API specifications, endpoint documentation, and payload schemas. (Currently empty/reserved for backend scaling).
- **`docs/guides/`**: Developer setup runbooks, contribution guidelines, and how-to manuals. (Currently empty/reserved).
- **`docs/research/`**: Exploratory research, prototyping notes, and pedagogical methodological studies. Use this to store outputs when analyzing abstract technological or pedagogical concepts.
  - `Building Grammar Skills for Vietnamese Kids.md`: Core cognitive framework for teaching English syntax to Vietnamese learners.
  - `Interactive ESL Tense App Design.md`: UX patterns mapping pedagogical theories to interactive mobile architectures.
  - `Phương pháp luận phần mềm học tập trẻ em.md`: Systemic methodology guiding retention through spaced repetition (SM-2 Lite) and active recall.
  - `ideation/tenses_game_ideation.md`: Concept brainstorming and scratchpad logic for tense-based gameplay loops.
- **Root Files (`docs/*.md`)**: High-level macro overlays.
  - `architecture.md`: The defining Wordlist Quiz Brownfield strategy and global integration boundaries.
  - `brownfield-architecture.md`: Deeper evaluation of the codebase layout, constraints, and transition from monolithic structs.

## Agile Dev Agentic Coding Skills

For this type of project, the Agentic SE workflow utilizes four core skills/personas:

1. **Business Analyst Persona (`agile-analyst`)**
2. **Architect Persona (`agile-architect`)**
3. **Engineer Persona (`agile-engineer`)**
4. **QA Persona (`agile-qa`)**

---

## Workflow Orchestration

### 1. Plan Mode Default

- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy

- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Self-Improvement Loop

- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done

- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)

- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing

- When given a bug report: just fix it. Don't ask for hand-holding

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duyhunghd6/wordlist-quiz](https://github.com/duyhunghd6/wordlist-quiz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
