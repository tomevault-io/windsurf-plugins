---
trigger: always_on
description: CC Version Guard — Tauri v2 Desktop Application
---

# AGENTS.md

CC Version Guard — Tauri v2 Desktop Application

Follows [MCAF](https://mcaf.managed-code.com/)

---

## Conversations (Self-Learning)

Learn the user's habits, preferences, and working style. Extract rules from conversations, save to "## Rules to follow", and generate code according to the user's personal rules.

**Update requirement (core mechanism):**

Before doing ANY task, evaluate the latest user message.
If you detect a new rule, correction, preference, or change → update `AGENTS.md` first.
Only after updating the file you may produce the task output.
If no new rule is detected → do not update the file.

---

## Rules to follow (Mandatory, no exceptions)

### Commands

- build: `npm run tauri build`
- dev: `npm run tauri dev`
- cargo build: `cargo build --release` (in src-tauri/)
- cargo test: `cargo test` (in src-tauri/)
- format: `cargo fmt` (in src-tauri/)
- lint: `cargo clippy` (in src-tauri/)

### Task Delivery (ALL TASKS)

- Read AGENTS.md and docs before planning
- Write multi-step plan before implementation
- Implement code and tests together
- Run builds yourself — do NOT ask user to execute them
- Summarize changes before marking complete

### Documentation (ALL TASKS)

- All docs live in `docs/`
- Update feature docs when behaviour changes
- Update ADRs when architecture changes
- Templates: `docs/templates/ADR-Template.md`, `docs/templates/Feature-Template.md`

### Testing

- Desktop GUI app — manual testing via `npm run tauri dev`
- Verify all wizard screens work before committing
- Test file system operations on real CapCut installations

### Project Structure

```
capcut_guard_tauri/
├── src/                         # Frontend (Vanilla JS)
│   ├── index.html              # Wizard screens
│   ├── styles.css              # Midnight Obsidian theme
│   ├── main.js                 # Wizard logic & Tauri IPC
│   └── assets/                 # Static assets
├── src-tauri/                  # Backend (Rust)
│   ├── src/
│   │   ├── main.rs             # Entry point
│   │   ├── lib.rs              # Tauri builder & command registration
│   │   └── commands/           # Tauri commands (modular)
│   │       ├── mod.rs
│   │       ├── scanner.rs      # Version scanning
│   │       ├── process.rs      # Process detection
│   │       ├── cleaner.rs      # Cache cleaning
│   │       ├── protector.rs    # File locking
│   │       └── switcher.rs     # Version switching
│   ├── Cargo.toml
│   └── tauri.conf.json
├── package.json
└── README.md
```

### Code Style

- Rust 2021 edition for backend
- Vanilla JavaScript (no framework) for frontend
- Use Phosphor Icons (web CDN) — never ASCII/Unicode symbols
- Follow 60-30-10 color rule for UI theming
- CSS variables for all design tokens

### External Resources (Mandatory)

- **Download Source**: ALWAYS use official ByteDance CDN URLs for legacy downloads
- **Icons**: Use Phosphor Icons from `unpkg.com/@phosphor-icons/web`


### UI/UX Framework: Laws of UX + design.json

> **Design System**: All visual tokens are defined in `design.json` (macOS 26 Tahoe).
> **Psychology Framework**: Laws of UX governs user experience decisions.

#### Design Token Source
- **Single Source of Truth**: `design.json` defines all colors, typography, spacing, radii, shadows, and animations
- **DO NOT** hardcode values — always reference CSS variables derived from design.json
- **DO NOT** deviate from design.json specifications

#### Laws of UX (Governing Principles)

**Perception & Aesthetics:**
- **Aesthetic-Usability Effect**: Users perceive beautiful design as more usable. Invest in visual polish.
- **Law of Prägnanz**: Simplify complex interfaces to their simplest form.

**Decision Making:**
- **Hick's Law**: Minimize choices per view. One primary CTA per screen.
- **Choice Overload**: Never present more than 5-7 options without filtering.

**Interaction Design:**
- **Fitts's Law**: Touch targets minimum 44×44px. CTAs should be large and easily acquired.
- **Doherty Threshold**: Provide feedback within 400ms to maintain user engagement.

**Grouping & Organization:**
- **Chunking**: Break content into digestible groups using glass panels.
- **Law of Proximity**: Related elements close together, unrelated elements apart.
- **Law of Common Region**: Use container borders to define content groups.
- **Law of Similarity**: Consistent styling for similar functions.

**Memory & Attention:**
- **Miller's Law**: Working memory holds 7±2 items. Don't overload.
- **Von Restorff Effect**: Make primary actions visually distinctive.
- **Serial Position Effect**: Important items at start and end of lists.
- **Selective Attention**: Guide focus, prevent distraction.

**Motivation & Progress:**
- **Zeigarnik Effect**: Show incomplete tasks/progress to drive completion.
- **Goal-Gradient Effect**: Progress indicators motivate action as users approach goals.
- **Peak-End Rule**: Endings matter. Polish success/error states.

**Familiarity:**
- **Jakob's Law**: Follow macOS native patterns users already know.
- **Mental Models**: Match design to user expectations.

**Efficiency:**
- **Tesler's Law**: Absorb complexity into the system, not the user.
- **Occam's Razor**: Remove unnecessary elements.
- **Pareto Principle**: 80% of users use 20% of features. Optimize the critical path.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zendevve/CCVG](https://github.com/Zendevve/CCVG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
