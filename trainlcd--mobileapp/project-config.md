---
trigger: always_on
description: This handbook defines how automation agents collaborate safely and effectively on the TrainLCD mobile application. Follow these instructions for every bot- or AI-assisted contribution, regardless of scope.
---

# Repository Guidelines

This handbook defines how automation agents collaborate safely and effectively on the TrainLCD mobile application. Follow these instructions for every bot- or AI-assisted contribution, regardless of scope.

## Operating Principles for Automation Agents

- **Honor instruction priority:** repository owners & maintainers → latest task prompt → this handbook → other documentation. Surface conflicting requirements immediately.
- **Preserve the working tree:** operate on the current snapshot, never discard user changes, and avoid destructive commands (`git reset --hard`, `git clean -fd`, etc.).
- **Favor minimal, auditable diffs:** prefer additive edits, keep formatting deterministic, and annotate non-obvious changes with concise comments.
- **Document reproducibility:** record every manual command you execute and note any local assumptions about environment variables or credentials.
- **Validate assumptions proactively:** confirm tool versions, workflow expectations, and environment needs instead of relying on cached knowledge.
- **Clarify uncertainty:** request guidance or leave TODO notes rather than guessing at intent.
- **Prioritize quality and performance over speed:** prefer well-structured, performant implementations over quick solutions. Take extra time to consider edge cases, optimize hot paths, and ensure code correctness rather than rushing to deliver.

## Standard Workflow

1. **Intake:** read the full issue, PR discussion, or prompt; restate deliverables and constraints before coding.
2. **Reconnaissance:** map relevant files with `rg`, `ls`, or `find`; review interfaces and existing patterns to plan compatible changes.
3. **Plan:** outline discrete steps, keep the plan updated as you progress, and expose blockers early.
4. **Implement:** use `apply_patch` for targeted edits, commit in small logical units, and avoid regenerating large files unless required.
5. **Validate:** run only the necessary commands (`npm run lint`, `npm test`, `npm run typecheck`, etc.) and capture summarized output.
6. **Document & Handoff:** update READMEs or docs when behavior changes, summarize modifications, list executed commands, and attach artifacts (logs, screenshots) before opening PRs.

## Repository Map

- `src/`: Expo React Native app code.
  - `src/components/`, `src/screens/`: UI components and screen containers.
  - `src/hooks/`, `src/store/`, `src/stacks/`: shared state, navigation, and composition hooks.
  - `src/lib/`, `src/providers/`, `src/config/`: integrations, context providers, configuration utilities.
  - `src/constants/`, `src/utils/`, `src/translation.ts`, `src/lineSymbolImage.ts`: constants, helpers, localization maps, and asset selectors.
  - `@types/`, `src/__mocks__/`, `src/__fixtures__/`, `test/`: global typings, reusable mocks, fixtures, and test helpers.
- `assets/`: static media (images, fonts, icons).
- `docs/`: human-facing documentation including changelog and incident notes.
- `utils/`: developer tooling scripts such as GraphQL codegen config.
- `android/`, `ios/`: native projects.

> The Cloudflare Workers backend (TTS via Azure Speech, feedback triage via Workers AI, review notifiers) has been moved out of this repository into the [TrainLCD/BFF](https://github.com/TrainLCD/BFF) monorepo. The former `functions/` directory no longer lives here.

## Tooling & Environment Expectations

- Target **Node.js 22.x** and **npm 10.x**.
- Run `npm install` when dependencies shift; avoid re-locking packages unless instructed.
- Metro cache issues: run `expo start --clear` only when debugging build failures and document the action.
- For native builds, rely on project scripts (`npm run android`, `npm run ios`).
- GraphQL codegen requires `GQL_API_URL` in `.env.local`; run `npm run gql:codegen` after document or schema updates.

## Build, Test & Development Commands

- `npm run start`: start the Expo Dev Client locally.
- `npm run android` / `npm run ios`: build native binaries.
- `npm run web`: run the web preview.
- `npm run lint`: execute Biome linting (`biome ci ./src` in CI).
- `npm run format`: apply Biome formatting fixes.
- `npm test`: run Jest in UTC; add `--watch` or `--runInBand` for debugging.
- `npm test -- --updateSnapshot`: refresh Jest snapshots when output diffs are intentional.
- `npm run typecheck`: enforce TypeScript constraints.
- `npm run gql:codegen`: regenerate generated GraphQL types.

## Coding Style & Naming Conventions

- `.editorconfig` enforces UTF-8, two-space indentation, single quotes, and ES5 trailing commas.
- Biome is authoritative; avoid `// biome-ignore` unless a rule is truly incompatible and document the rationale inline.
- Components → PascalCase (`StationBanner.tsx`); hooks → `use*` (`useStationFeed.ts`); Jotai atoms → `store/atoms/*.ts`; GraphQL operations → `FeatureVerbQuery`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TrainLCD/MobileApp](https://github.com/TrainLCD/MobileApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
