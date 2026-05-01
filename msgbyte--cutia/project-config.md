---
trigger: always_on
description: **This file provides an index of exported functions, types, interfaces, classes, and constants in your codebase.**
---

---
alwaysApply: true
---

# video-editor-oss Codebase Index

**This file provides an index of exported functions, types, interfaces, classes, and constants in your codebase.**

Updated in real-time by Twiggy. Use this to discover existing utilities and avoid duplicating code.

## How to Use

When implementing new features:
1. Check if similar functionality already exists
2. Reuse existing types and utilities
3. Understand the API surface of your codebase

```typescript
## apps/web/src/constants

editor-constants.ts
  export const PLATFORM_LAYOUTS: Record<TPlatformLayout, string>
  export const PANEL_CONFIG

export-constants.ts
  export const DEFAULT_EXPORT_OPTIONS
  export const EXPORT_MIME_TYPES

font-constants.ts
  export interface FontOption {
    value: string
    label: string
    category: "system" | "google" | "custom"
    weights?: number[]
    hasClassName?: boolean
  }
  export const FONT_OPTIONS: FontOption[]
  export const DEFAULT_FONT
  export type FontFamily = (typeof FONT_OPTIONS)[number]["value"]
  export const getFontByValue = (value: string): FontOption | undefined => ...
  export const getGoogleFonts = (): FontOption[] => ...
  export const getSystemFonts = (): FontOption[] => ...

language-constants.ts
  export const LANGUAGES

project-constants.ts
  export const DEFAULT_CANVAS_PRESETS: TCanvasSize[]
  export const FPS_PRESETS
  export const BLUR_INTENSITY_PRESETS: { label: string; value: number }[]
  export const DEFAULT_CANVAS_SIZE: TCanvasSize
  export const DEFAULT_FPS
  export const DEFAULT_BLUR_INTENSITY
  export const DEFAULT_COLOR

site-constants.ts
  export const SITE_URL
  export const SITE_INFO
  export type ExternalTool = {
  	name: string;
  	description: string;
  	url: string;
  	icon: React.ElementType;
  }
  export const EXTERNAL_TOOLS: ExternalTool[]
  export const DEFAULT_LOGO_URL
  export const SOCIAL_LINKS
  export type Sponsor = {
  	name: string;
  	url: string;
  	logo: string;
  	description: string;
  }
  export const SPONSORS: Sponsor[]

stickers-constants.ts
  export const STICKER_CATEGORIES
  export const STICKER_CATEGORY_CONFIG: Record<
  	(typeof STICKER_CATEGORIES)[number],
  	string | undefined
  >

text-constants.ts
  export const DEFAULT_TEXT_ELEMENT: Omit<TextElement, "id">

timeline-constants.tsx
  export const TRACK_COLORS: Record<TrackType, { background: string }>
  export const TRACK_HEIGHTS: Record<TrackType, number>
  export const TRACK_GAP
  export const TIMELINE_CONSTANTS
  export const DEFAULT_TIMELINE_VIEW_STATE: TTimelineViewState
  export const TRACK_ICONS: Record<TrackType, React.ReactNode>

transcription-constants.ts
  export const TRANSCRIPTION_LANGUAGES
  export const TRANSCRIPTION_MODELS: TranscriptionModel[]
  export const DEFAULT_TRANSCRIPTION_MODEL: TranscriptionModelId
  export const DEFAULT_CHUNK_LENGTH_SECONDS
  export const DEFAULT_STRIDE_SECONDS
  export const DEFAULT_WORDS_PER_CAPTION
  export const MIN_CAPTION_DURATION_SECONDS

## apps/web/src/core

index.ts
  export class EditorCore {
    instance: EditorCore | null
    command: CommandManager
    playback: PlaybackManager
    timeline: TimelineManager
    scenes: ScenesManager
    project: ProjectManager
    media: MediaManager
    renderer: RendererManager
    save: SaveManager
    audio: AudioManager
    selection: SelectionManager
    static getInstance(): EditorCore
    static reset(): void
  }

## apps/web/src/hooks

use-editor.ts
  export function useEditor(): EditorCore

use-file-upload.ts
  export function useFileUpload({
  	accept,
  	multiple,
  	onFilesSelected,
  }: UseFileUploadOptions = {})

use-infinite-scroll.ts
  export function useInfiniteScroll({
  	onLoadMore,
  	hasMore,
  	isLoading,
  	threshold = 200,
  	enabled = true,
  }: UseInfiniteScrollOptions)

use-keybindings.ts
  export function useKeybindingsListener()
  export function useKeybindingDisabler()

use-keyboard-shortcuts-help.ts
  export interface KeyboardShortcut {
    id: string
    keys: string[]
    description: string
    category: string
    action: TAction
    icon?: React.ReactNode
  }
  export function useKeyboardShortcutsHelp()

use-mobile.ts
  export function useIsMobile()

use-raf-loop.ts
  export function useRafLoop(callback: ({ time }: { time: number }) => void)

use-reveal-item.ts
  export function useRevealItem(
  	highlightId: string | null,
  	onClearHighlight: () => void,
  	highlightDuration = 1000,
  )

use-sound-search.ts
  export function useSoundSearch({
  	query,
  	commercialOnly,
  }: {
  	query: string;
  	commercialOnly: boolean;
  })

## apps/web/src/hooks/actions

use-action-handler.ts
  export function useActionHandler(
  	action: A,
  	handler: TActionFunc<A>,
  	isActive: TActionHandlerOptions,
  )

use-editor-actions.ts
  export function useEditorActions()

## apps/web/src/hooks/storage

use-local-storage.ts
  export function useLocalStorage({
  	key,
  	defaultValue,
  }: {
  	key: string;
  	defaultValue: T;
  }): [
  	T,
  	({ value }: { value: T | ((previousValue: T) => T) }) => void,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [msgbyte/cutia](https://github.com/msgbyte/cutia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
