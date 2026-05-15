---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TypeScript monorepo that splits LLM-generated markdown text into WhatsApp-friendly chat message chunks. The core algorithm intelligently breaks long text at natural boundaries (questions, periods, lists, markdown sections) while preserving URLs, numbers, emails, abbreviations, parenthetical expressions, and Spanish punctuation.

The primary use case is Latin American e-commerce customer service over WhatsApp, where LLMs generate long Spanish responses about products (Nike shoes, clothing, etc.) that need to be split into readable chat messages.

## Commands

```bash
npm install                  # Install all workspace dependencies
npm run build                # Build all packages
npm run build:core           # Build core package only
npm test                     # Run all tests
npm run test:core            # Run core tests only
npm run typecheck            # Type check all packages (tsc -b)
npm run lint                 # ESLint
npm run format               # Prettier
npm run check                # Format + lint + typecheck

# Run a single test file
cd packages/core && NODE_OPTIONS='--experimental-vm-modules' npx jest --testPathPattern="splitChatText"

# Watch mode for core tests
cd packages/core && NODE_OPTIONS='--experimental-vm-modules' npx jest --watch
```

Note: `NODE_OPTIONS='--experimental-vm-modules'` is required because the project uses ESM modules with ts-jest.

## Architecture

**Monorepo structure:** npm workspaces with `packages/*`. Currently only `packages/core` (`@daviddh/llm-markdown-whatsapp/core`) exists. The root `tsconfig.json` references additional packages (redis, e2e/*) that are not yet present.

**Core package entry point:** `packages/core/src/index.ts` re-exports `splitChatText` from `packages/core/src/chatSplit/index.ts`, which re-exports from `splitChatText.ts`. This is the single public API function.

### splitChatText Pipeline

`splitChatText(text)` in `packages/core/src/chatSplit/splitChatText.ts` is the orchestrator. It accepts `string | null | undefined` and returns `string[]`.

**1. Pre-processing** (`preProcessText`):
- `removePeriodsAfterURLs` (`urlNormalization.ts`): Replaces `.` after URLs with `\n` (URLs never end with periods)
- `normalizeInlineNumberedList` (`listNormalization.ts`): Detects inline patterns like `1. X 2. Y 3. Z` and adds line breaks between items. Skips already-formatted lists. Handles both colon-preceding and question-preceding patterns.
- `normalizeInlineProductCardList` (`listNormalization.ts`): Detects inline product cards (with `🛍️` or markdown formatting + emoji indicators) and adds line breaks before each card, before emoji indicators within cards, and before trailing questions.

**2. Main loop** — iterates while `remainingText !== ''`, trying processors in priority order. First match wins, remaining text is re-evaluated from the top.

Processor groups (in `splitChatText.ts`):

- **`runIntroAndListProcessors`** (highest priority):
  - `processIntroWithList` (`splitProcessors.ts`): Matches `intro:` + newline + list start (`\d. ` or `- `). If intro < 150 chars, splits after intro. Handles "Puedes responder con:" pattern specially.
  - `processQuestionWithList` (`splitProcessors.ts`): Matches `question?\n` + numbered list. Keeps together as one chunk if total < 250 chars and >= 2 list items.
  - `processIntroWithLongParagraphs` (`splitProcessors.ts`): Matches `intro:\n` + paragraph > 150 chars. Splits after intro.

- **`runContentStructureProcessors`**:
  - `processProductCardLists` (`productCardProcessor.ts`): Detects product cards by emoji pattern (`\d. 🛍️`) or markdown pattern (`\d. *Title*` + emoji indicators). Extracts intro, splits each card into its own chunk (removing the `\d.` prefix), and separates trailing questions from the last card via `extractTrailingQuestion`.
  - `processListSection` (`listProcessor.ts`): Uses `findListSection` (`sections.ts`) to detect numbered or bullet lists. Splits numbered lists per-item if items > 150 chars or average > 70 chars with <= 3 items. Splits bullet lists per-item only if items > 150 chars. Otherwise keeps the entire list as one chunk.
  - `processLongParagraphsAfterIntro` (`paragraphProcessor.ts`): `intro:\n` followed by multiple paragraphs where at least one > 150 chars. Splits after intro.
  - `processLongParagraphSequence` (`paragraphProcessor.ts`): First paragraph > 150 chars with multiple lines. Splits after first paragraph (unless followed by "question with options" pattern).

- **`runFormattingProcessors`**:
  - `processMarkdownSection` (`paragraphProcessor.ts`): Uses `findMarkdownSection` (`sections.ts`) to detect `*Header*` or `_Header_` followed by content until next `\n\n` or end. Splits at section boundaries.
  - `processSectionBreaks` (`breakProcessor.ts`): Splits at `\n\n` (double newline) if > 50 chars before the break. Does NOT split if: before ends with `?` and after has short intro + bullets, or before ends with "Puedes responder con:" + bullets, or after has question-with-options pattern. Also checks for markdown headers after break and long paragraphs before break.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daviddominguezh/llm-markdown-whatsapp](https://github.com/daviddominguezh/llm-markdown-whatsapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
