---
trigger: always_on
description: The repository is a Flutter application that follows the Flutter app architecture guidance with explicit layering, MVVM-style presentation, dependency injection via `get_it`, and modular agent resources under `.agents/`.
---

# AGENTS.md

## Project Overview

The repository is a Flutter application that follows the Flutter app architecture guidance with explicit layering, MVVM-style presentation, dependency injection via `get_it`, and modular agent resources under `.agents/`.

The project includes:

- **Agents** - custom agent mode definitions under `.agents/agents/`
- **Instructions** - file-pattern-based coding guidance under `.agents/instructions/`
- **Skills** - reusable specialized workflows under `.agents/skills/`
- **Templates** - reusable markdown assets under `.agents/templates/`
- **Shared agent instructions** - repository-wide guidance in `.agents/instructions.md`
- **Flutter application code** - app source under `lib/`, tests under `test/`, and architecture docs under `docs/`

This repository is not a generic agent marketplace. The `.agents/` directory exists to support AI-assisted work on this specific Flutter codebase.

## Repository Structure

```text
.
├── .agents/
│   ├── agents/         # Custom agent mode definitions (.md with frontmatter)
│   ├── instructions/   # File-pattern instructions (.md with frontmatter)
│   ├── skills/         # Skill folders (each with SKILL.md)
│   ├── templates/      # Reusable markdown templates
│   ├── README.md       # .agents directory overview
│   └── instructions.md # Shared repository-wide agent instructions
├── docs/               # Architecture and supporting documentation
├── lib/                # Flutter application source
│   ├── data/           # Repositories and services
│   ├── domain/         # Models and use cases
│   ├── routing/        # go_router configuration
│   ├── ui/             # Screens, widgets, and view models
│   ├── utils/          # Shared utilities
│   ├── injector.dart   # Dependency injection
│   └── main.dart       # App bootstrap
├── test/               # Tests by layer
├── web/                # Web entry assets
├── android/            # Android host app
├── ios/                # iOS host app
├── pubspec.yaml        # Dart/Flutter dependencies
└── README.md           # Project overview
```

## Setup Commands

```bash
# Install Dart and Flutter dependencies
flutter pub get

# Run the app with environment values
flutter run --dart-define-from-file=.env

# Run unit and widget tests
flutter test

# Analyze the project
flutter analyze

# Format Dart files
dart format lib test .agents
```

If dependency resolution changes, prefer rerunning `flutter pub get` rather than editing generated lock state manually.

## Development Workflow

### Working with Agents, Instructions, and Skills

All resources under `.agents/` should be lightweight, explicit, and aligned with the actual architecture of this repository.

#### Agent Files (`.agents/agents/*.md`)

- Must have markdown frontmatter
- Must include a `description` field
- Should include a human-readable `name`
- May include a `tools` array when the consuming platform supports it
- File names should be lower case with words separated by hyphens
- Agent content should reflect this repository's Flutter architecture, not generic unrelated stacks

#### Instruction Files (`.agents/instructions/*.md`)

- Must have markdown frontmatter
- Must include a non-empty `description` field
- Must include an `applyTo` field with file patterns
- File names should be lower case with words separated by hyphens
- Instructions should map to real file locations in this repository

#### Skills (`.agents/skills/*/SKILL.md`)

- Each skill lives in its own folder
- `SKILL.md` must include frontmatter
- `name` should match the folder name in lowercase hyphenated form
- `description` should describe when the skill should be used
- Skills should stay focused on one package, layer, or workflow
- Any bundled assets should be referenced from `SKILL.md`

### Adding New Resources

When adding a new agent, instruction, or skill:

1. place it in the correct `.agents/` subdirectory
2. add proper frontmatter
3. align naming with the existing lower-case hyphenated convention
4. make the content specific to this codebase and architecture
5. verify referenced files and paths actually exist

### Adding New Instructions

For instructions under `.agents/instructions/`:

1. create a new `.md` file with a descriptive lower-case hyphenated name
2. add `description` and `applyTo` in frontmatter
3. target real paths such as:
   - `lib/ui/**`
   - `lib/domain/**`
   - `lib/data/**`
4. keep the instruction focused on one layer or concern

### Adding New Skills

For skills under `.agents/skills/`:

1. create a new folder with a lower-case hyphenated name
2. add `SKILL.md`
3. include `name` and `description` in frontmatter
4. keep the skill concise and task-oriented
5. reference only the files and workflows relevant to the skill

## Architecture Rules

This project follows these architectural boundaries:

- `ui -> view_models -> use_cases -> repositories -> services`
- `lib/data/` owns infrastructure-facing concerns
- `lib/domain/` owns models and use cases
- `lib/ui/` owns widgets, screens, and view models
- `lib/routing/` owns navigation
- `lib/injector.dart` owns DI wiring

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rkpontes/flutter_official_architecture](https://github.com/rkpontes/flutter_official_architecture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
