---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development

- **Start dev server**: `npm run dev` or `bun dev` - Runs on port 8082
- **Build production**: `npm run build` - Creates optimized build in `dist/`
- **Build development**: `npm run build:dev` - Development mode build
- **Preview build**: `npm run preview` - Preview production build locally
- **Lint code**: `npm run lint` - Run ESLint checks (MVP config: warnings only)
- **Format code**: `npm run format` - Format source files with Prettier
- **Check formatting**: `npm run format:check` - Check Prettier formatting without changes

### Testing

- **Run tests**: `npm run test` - Interactive test watcher with Vitest
- **Run tests once**: `npm run test:run` - Single test run for CI/CD
- **Test UI**: `npm run test:ui` - Visual test interface
- **Test setup**: Vitest + React Testing Library + jsdom environment
- **Mocks included**: Web Audio API, IntersectionObserver, ResizeObserver, matchMedia

## Architecture

### Technology Stack

- **Framework**: React 18.3.1 with TypeScript
- **Build Tool**: Vite with SWC for fast compilation
- **Styling**: Tailwind CSS with custom glassmorphism design system
- **UI Components**: Radix UI primitives wrapped in shadcn/ui pattern
- **Voice AI**: @elevenlabs/react SDK v0.12.1, OpenAI Realtime API, xAI Realtime API, Ultravox SDK, Vapi SDK, Retell SDK, Google Gemini Live API
- **Animations**: Framer Motion for smooth transitions
- **State**: React Context (theme), Tanstack Query (server state), custom hooks

### Project Structure

```
src/
|-- components/          # UI components
|   |-- voice/          # Voice-specific components
|   |   |-- VoiceButton.tsx # Voice interaction button
|   |   |-- VoiceStatus.tsx # Connection status display
|   |   |-- VoiceVisualizer.tsx # Audio visualization component
|   |   |-- VoiceSelector.tsx # Voice selection dropdown (Phase 02)
|   |   |-- ConversationPanel.tsx # Real-time transcript display (Phase 02)
|   |   |-- MessageBubble.tsx # Individual message component (Phase 02)
|   |   `-- FunctionCallIndicator.tsx # Function execution UI (Phase 02)
|   |-- providers/      # Provider-specific components
|   |   |-- ElevenLabsProvider.tsx # ElevenLabs voice interface
|   |   |-- OpenAIProvider.tsx # OpenAI Realtime interface
|   |   |-- XAIProvider.tsx # xAI Grok interface
|   |   |-- UltravoxProvider.tsx # Ultravox voice interface (Phase 04)
|   |   |-- VapiProvider.tsx # Vapi voice interface (Phase 05)
|   |   |-- RetellProvider.tsx # Retell voice interface (Phase 06)
|   |   |-- GeminiProvider.tsx # Gemini Live voice interface (Phase 00-Session 04)
|   |   `-- GeminiEmptyState.tsx # Gemini empty state component
|   |-- tabs/           # Tab navigation
|   |   |-- ProviderTabs.tsx # Provider tab container
|   |   `-- ProviderTab.tsx # Individual tab component
|   |-- BackgroundEffects.tsx # Dynamic background animations
|   |-- HeroSection.tsx # Landing page hero section
|   |-- VoiceEnvironment.tsx # 3D voice environment
|   |-- ConfigurationModal.tsx # Settings modal
|   `-- ui/             # 50+ shadcn/ui components
|-- hooks/              # Business logic
|   |-- useAccessibility.ts # Accessibility features
|   |-- useReconnection.ts # WebSocket reconnection with backoff (Phase 02)
|   |-- useVapiVoice.ts # Vapi voice hook (Phase 05)
|   |-- useRetellVoice.ts # Retell voice hook (Phase 06)
|   |-- useGeminiVoice.ts # Gemini Live voice hook (Phase 00-Session 04)
|   |-- use-mobile.tsx  # Mobile detection hook
|   `-- use-toast.ts    # Toast notifications
|-- pages/              # Route components
|   |-- Index.tsx       # Main app page (uses env vars for Agent ID)
|   `-- NotFound.tsx    # 404 error page
|-- test/               # Test infrastructure (215+ tests)
|   |-- setup.ts        # Test configuration and mocks
|   |-- useReconnection.test.ts # Reconnection logic tests
|   |-- voiceConfig.test.ts # Voice selection tests
|   |-- ConversationPanel.test.tsx # Transcript tests
|   `-- ... (14 test files total)
|-- contexts/           # Global state
|   |-- ThemeContext.tsx # Dark/light theme management
|   |-- VoiceContext.tsx # ElevenLabs voice state
|   |-- XAIVoiceContext.tsx # xAI voice state with reconnection (Phase 02)
|   |-- OpenAIVoiceContext.tsx # OpenAI voice state with reconnection (Phase 02)
|   |-- UltravoxVoiceContext.tsx # Ultravox voice state (Phase 04)
|   |-- GeminiVoiceContext.tsx # Gemini Live voice state (Phase 00-Session 04)
|   `-- ProviderContext.tsx # Active provider selection
`-- lib/                # Utilities
    |-- utils.ts        # Helper functions
    |-- audio/          # Audio processing (Phase 02)
    |   `-- audioUtils.ts # PCM encoding, base64 conversion
    |-- gemini/          # Gemini Live infrastructure (Phase 00-Session 04)
    |   |-- genai-live-client.ts # WebSocket client for Gemini Live API
    |   |-- audio-recorder.ts # PCM audio capture for Gemini
    |   |-- audio-streamer.ts # PCM audio playback for Gemini
    |   `-- config.ts # Gemini configuration constants
    `-- tools/          # Function calling (Phase 02)
        `-- toolDefinitions.ts # Weather, time, calculator tools
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moshehbenavraham/Voice-Agent-PuPuPlatter](https://github.com/moshehbenavraham/Voice-Agent-PuPuPlatter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
