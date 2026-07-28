---
trigger: always_on
description: Guidance for Claude Code (and any AI assistant / contributor) working in this
---

# CLAUDE.md

Guidance for Claude Code (and any AI assistant / contributor) working in this
repository. This file is **self-contained** — architecture, conventions, and
workflow all live here. Keep it up to date as the codebase evolves.

---

## 1. What this is

**Obsidian Voice** — a text-to-speech plugin for Obsidian that reads notes
aloud with an audiobook-style player. It supports **five providers** (AWS Polly,
ElevenLabs, OpenAI, Google Cloud, Azure Speech) and runs on **desktop and
mobile** (iOS / Android). Users bring their own provider credentials; nothing is
proxied through a third party.

### Tech stack

- **TypeScript** (strict) — primary language.
- **Obsidian API** (`obsidian` 1.13.x) — plugin framework.
- **esbuild** — bundler (`esbuild.config.mjs`, outputs `main.js`).
- **ESLint** + `eslint-plugin-obsidianmd` — linting and Obsidian-specific rules.
- **Prettier** — formatting.
- **Jest** (`jest-environment-node`) + `ts-jest` — tests.
- **unified / remark** — Markdown → AST for the content pipeline.
- **@aws-sdk/client-polly** — AWS Polly client (other providers use `requestUrl`).
- **semantic-release** — automated versioning/changelog from commit messages.

---

## 2. Architecture

### Entry point & orchestration

- `src/main.ts` re-exports the plugin class — the manifest's entry.
- `src/utils/VoicePlugin.ts` (`class Voice extends Plugin`) is the heart: it
  loads settings, builds the active provider via the factory, and wires the
  collaborators below. It owns lifecycle, commands, the ribbon icon, the player
  view registration, and the "What's New" modal.

The `Voice` instance wires together:

| Collaborator       | File                          | Responsibility                                          |
| ------------------ | ----------------------------- | ------------------------------------------------------- |
| Speech provider    | `service/*`                   | Synthesis + playback for the selected engine.           |
| `TextSpeaker`      | `utils/TextSpeaker.ts`        | Orchestrates "read the active note" (pipeline → speak). |
| `MarkdownHelper`   | `utils/MarkdownHelper.ts`     | Reads the active note / selection from Obsidian.        |
| `IconEventHandler` | `utils/IconEventHandler.ts`   | Status-bar controls, ribbon state, save orchestration.  |
| `MobileControlBar` | `utils/MobileControlBar.ts`   | Touch control bar (mobile only).                        |
| `VoicePlayerView`  | `ui/VoicePlayerView.ts`       | The audiobook-style player pane.                        |
| `HotkeySettings`   | `settings/HotkeySettings.ts`  | Registers the ~16 hotkey commands.                      |
| `VoiceSettingTab`  | `settings/VoiceSettingTab.ts` | The settings UI.                                        |

### Provider layer (`src/service/`)

All providers implement one interface so the rest of the plugin is
**provider-agnostic**:

- `SpeechProvider.ts` — the interface (synthesis, playback, voice, caching,
  credential validation, operation lifecycle, progress/error callbacks).
- `BaseSpeechService.ts` — the abstract base holding **all shared logic**
  (playback, the `<audio>` element, rewind/forward, speed, per-note audio
  caching keyed by file path, operation/request lifecycle). Subclasses only
  implement what actually differs: `speak()`, `validateCredentials()`,
  `updateCredentials()`, `getVoiceOptions()`, and `inputFormat`.
- Concrete services: `AwsPollyService`, `AzureSpeechService`, `GoogleTtsService`,
  `ElevenLabsService`, `OpenAiSpeechService`.
- `SpeechProviderFactory.ts` — `createSpeechProvider(settings)` builds the
  provider chosen in settings and applies rewind/forward prefs.
- `textChunker.ts` — splits long text for the text-input providers.
- `voiceCatalog.ts` — **pure** helpers (unit-tested) to map a provider's fetched
  voice list into `VoiceOption[]` and group it by language for the picker.
  Azure uses it: "Test Credentials" fetches `/voices/list`, the result is cached
  in `settings.azureVoiceCatalog`, and `getVoiceOptions()` returns it (the
  hardcoded `AZURE_VOICES` is the fallback). The player renders the catalog as
  `<optgroup>`s grouped by language.

`inputFormat` selects which content pipeline feeds the provider:

- **`"ssml"`** → AWS Polly, Azure Speech, Google Cloud.
- **`"text"`** → ElevenLabs, OpenAI.

### Content pipeline (`src/processors/`)

Markdown is parsed with remark and transformed before synthesis. Two
orchestrators pick the path by `inputFormat`:

- `MarkdownToSSMLProcessor.ts` — for SSML providers. Stages live in
  `pipeline/`: **Clean** (`CleanProcessor`) → **Enhance** (`EnhanceProcessor`,
  adds prosody/breaks for headings/emphasis) → **XML escape**
  (`XmlEscapeProcessor`) → **Serialize** (`SSMLSerializer`) → **Validate**
  (`SSMLValidator`) → **Chunk** (`SSMLChunker`, keeps chunks within provider
  limits). `acronyms.ts` handles spell-out.
- `MarkdownToTextProcessor.ts` — for text providers; produces plain spoken text
  via `TextSerializer`.
- `config/DefaultConfig.ts` — default pipeline options; `types/ProcessorTypes.ts`
  and `types/SSMLNodes.ts` hold the shared types/AST node shapes.

`TextSpeaker.speakText()` is the funnel: read note → run the right pipeline →
`provider.speak(content, speed, filePath)`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisurf/obsidian-voice](https://github.com/chrisurf/obsidian-voice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
