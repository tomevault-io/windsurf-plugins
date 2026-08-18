---
trigger: always_on
description: Obsidian plugin (TypeScript + React) that records voice, transcribes audio (OpenAI Whisper or AssemblyAI), summarizes via LLM (LangChain + OpenAI), and writes structured Markdown notes into the vault. Built with esbuild, formatted with Biome.
---

# Obsidian-Scribe Agent Instructions

## Project Overview

Obsidian plugin (TypeScript + React) that records voice, transcribes audio (OpenAI Whisper or AssemblyAI), summarizes via LLM (LangChain + OpenAI), and writes structured Markdown notes into the vault. Built with esbuild, formatted with Biome.

## Available commands

### After writing code - Format, Lint, and Build

After writing code, always run these commands before completing the task:

1. `npm run format:write` — auto-format with Biome
2. `npm run lint:write` — lint with Biome (use `npm run lint:fix` to auto-apply safe fixes)
3. `npm run build:prod` — typecheck + production build

All three must pass cleanly. Fix any errors before finishing.

Entry: `main.ts` → re-exports default class from `src/index.ts`. Output: `build/main.js` (CJS bundle).

### Commands to never run
`npm run dev            # Never run this command.  You will get stuck.
`npm run update-all-deps # Never run this command.

## Project Structure

```
main.ts                          # Entrypoint, re-exports plugin class
src/
  index.ts                       # ScribePlugin class — the god object, all orchestration
  styles.css                     # Global plugin CSS
  audioRecord/audioRecord.ts     # MediaRecorder wrapper
  commands/commands.ts           # Obsidian command registrations
  ribbon/ribbon.ts               # Ribbon icon + dropdown menu
  modal/
    scribeControlsModal.tsx      # Modal class + React root mount
    components/                  # Modal UI components
      options/                   # Modal option sub-panels
    icons/icons.tsx              # SVG icon components
  settings/
    settings.tsx                 # Settings tab class, interfaces, defaults, React root mount
    GeneralSettingsTab.tsx        # General settings page
    ProviderSettingsTab.tsx       # AI provider settings page
    components/                  # Reusable settings UI controls
    hooks/useSettingsForm.tsx     # register()-pattern hook for settings controls
    provider/SettingsFormProvider.tsx  # React context for settings state
  util/
    assemblyAiUtil.ts            # AssemblyAI transcription
    audioDataToChunkedFiles.ts   # Audio decode, mono conversion, WAV chunking
    consts.ts                    # Shared enums (LanguageOptions, RECORDING_STATUS)
    filenameUtils.ts             # Date-based filename formatting
    fileUtils.ts                 # Vault file operations (save, create, rename, append)
    mimeType.ts                  # MIME type detection and mapping
    openAiUtils.ts               # OpenAI Whisper transcription + LangChain summarization
    pathUtils.ts                 # Obsidian path resolution
    textUtil.ts                  # Mermaid extraction, JSON key sanitization
    useDebounce.tsx              # Debounce hook
```

### Folder Conventions
- **Feature-folder grouping**: each domain (modal, settings, audioRecord, commands, ribbon) gets its own folder
- **Flat util folder**: one concern per file in `src/util/`
- **Components nest under parent feature**: `modal/components/`, `settings/components/`
- **No barrel/index re-exports**: every consumer imports directly from the source file path

## Naming Conventions

| Category | Convention | Examples |
|---|---|---|
| Local variables | camelCase | `audioBuffer`, `scribeOptions`, `baseFileName` |
| Booleans | `is`/`has`/`should` prefix | `isActive`, `isPaused`, `hasOpenAiApiKey` |
| Constants | UPPER_SNAKE_CASE | `DEFAULT_SETTINGS`, `MAX_CHUNK_SIZE` |
| Enums | PascalCase name | `TRANSCRIPT_PLATFORM`, `LanguageOptions`, `LLM_MODELS` |
| Interfaces/Types | PascalCase, no `I` prefix | `ScribeState`, `ScribeOptions`, `ScribePluginSettings` |
| Handlers | `handle` prefix | `handleStart`, `handlePauseResume`, `handleComplete` |
| CSS classes | `scribe-` prefix, kebab-case | `scribe-modal`, `scribe-btn-start` |

## Export Patterns

- **Plugin class**: `export default class` in `src/index.ts`
- **Page-level components & hooks**: `export default function` (`GeneralSettingsTab`, `useSettingsForm`, `AudioDeviceSettings`)
- **Utility functions & sub-components**: named `export function` (`handleCommands`, `ModalRecordingButtons`, `SettingsToggle`)
- **Constants & display components**: named `export const` (`DEFAULT_SETTINGS`, `SettingsItem`, `AiModelSettings`)
- **Enums**: named `export enum` (`TRANSCRIPT_PLATFORM`, `LLM_MODELS`, `LanguageOptions`)
- **Types/Interfaces**: named `export interface` / `export type`
- No barrel re-exports. Import directly from source paths.

## Code Style

- **Formatter**: Biome — 2-space indent, single quotes, semicolons, trailing commas
- **File extensions**: `.ts` for pure logic, `.tsx` for anything with JSX
- **Imports**: `import type` for type-only imports; `src/` base path alias for cross-feature imports, relative `./` within the same feature
- **Async**: `async/await` throughout
- **Error handling**: `try/catch/finally` with `new Notice(...)` for user feedback, `console.error` for dev logs
- **Comments**: sparse — JSDoc for attribution, occasional inline explanations. No pervasive function docs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikealicea/obsidian-scribe](https://github.com/mikealicea/obsidian-scribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-18 -->
