---
trigger: always_on
description: Story Game AI is a Next.js 15.5.2 application that creates interactive AI-powered story games. It uses Google AI for story and image generation, React 19 for the UI, Tailwind CSS 4 for styling, and TypeScript for type safety.
---

# Story Game AI

Story Game AI is a Next.js 15.5.2 application that creates interactive AI-powered story games. It uses Google AI for story and image generation, React 19 for the UI, Tailwind CSS 4 for styling, and TypeScript for type safety.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Quick Start
1. `pnpm install` (20 seconds)
2. `pnpm dev` (starts in 1 second)
3. Open http://localhost:3000
4. Verify the Story Game AI interface loads with sidebar settings and "Start Game" button

## Working Effectively

### Bootstrap, build, and test the repository:
- `pnpm install` -- takes 20 seconds. Installs 429 packages.
- `pnpm lint` -- takes <1 second. ALWAYS run before committing.
- `pnpm build` -- takes 30 seconds. NEVER CANCEL. Set timeout to 90+ minutes.
  - **CRITICAL**: Build WILL FAIL due to Google Fonts network restrictions with error "Failed to fetch `Geist` from Google Fonts". This is expected in restricted environments.
  - **WORKAROUND**: Temporarily comment out font imports in `src/app/layout.tsx` if production build is required.
- `pnpm dev` -- takes 1 second to start, 15 seconds for first compile. NEVER CANCEL. Set timeout to 30+ minutes.
- `pnpm start` -- starts production server in <1 second.

### Run the application:
- **Development**: `pnpm dev` then open http://localhost:3000
- **Production**: `pnpm build && pnpm start` then open http://localhost:3000
- **CRITICAL**: Application requires GOOGLE_GENERATIVE_AI_API_KEY environment variable to generate stories. Without it, the game will show error messages but the UI will still function.

### Environment Setup:
- **Required**: Node.js 18+ (tested with 20.19.4)
- **Required**: pnpm (install with `npm install -g pnpm`)
- **Optional**: Google AI API key from https://ai.dev for full functionality

## Validation

### Always manually validate any new code changes:
1. **ALWAYS run `pnpm lint`** -- Must pass with no errors before committing
2. **Start dev server** -- `pnpm dev` and verify http://localhost:3000 loads
3. **Test UI functionality**:
   - Verify sidebar story settings are editable
   - Confirm "Start Game" button works (will show error without API key, this is expected)
   - Check responsive design on different screen sizes
4. **Build validation** -- Run `pnpm build` to test production build capability
   - **EXPECTED**: Build will fail with "Failed to fetch `Geist` from Google Fonts" - this is normal in restricted environments
   - Document this as "Build fails due to expected network restrictions" in your changes

### NEVER CANCEL BUILD OR TEST COMMANDS:
- Builds may take 30+ minutes in some environments
- Always set timeouts to 90+ minutes for build commands
- Always set timeouts to 30+ minutes for development server startup

### Functional Testing Scenarios:
After making changes, always test these scenarios:
1. **Application Startup**: Navigate to http://localhost:3000, verify welcome screen appears with dark theme
2. **Story Configuration**: Modify settings in sidebar (genre, style, narrator role, initial situation, invite phrases, image style)
3. **Game Start Flow**: Click "Start Game" and verify conversation interface appears
4. **Error Handling**: Verify graceful degradation when API key is missing (shows error but UI remains functional)
5. **Responsive Design**: Test on different viewport sizes using browser dev tools
6. **Take Screenshot**: Document any UI changes for stakeholders

## Common Tasks

### Repository Structure:
```
src/
  app/
    api/
      generate-image/     # Image generation API endpoint
      generate-story/     # Story generation API endpoint
    components/           # Game-specific components
    hooks/               # Custom React hooks
    layout.tsx           # Main layout with fonts
    page.tsx            # Home page component
  components/            # Shared UI components
    ui/                 # UI primitives (buttons, inputs, etc.)
  lib/                  # Utility functions, constants, prompts
  types/                # TypeScript type definitions
```

### Key Components:
- **src/app/components/**: Game-specific components (input, loader, message, sidebar, story config)
- **src/components/**: Shared components (conversation, image, reasoning, etc.)
- **src/components/ui/**: UI primitives (avatar, badge, button, input, etc.)
- **src/lib/**: Utility functions, constants, and prompts
- **src/types/**: Type definitions for game and settings

### API Routes:
- **/api/generate-story/**: Handles story generation requests using Google AI
- **/api/generate-image/**: Handles image generation requests using Google AI

### Important Files:
- **biome.json**: Linting and formatting configuration
- **package.json**: Dependencies and scripts (use pnpm, not npm)
- **next.config.ts**: Next.js configuration
- **tailwind.config.js**: Tailwind CSS configuration
- **tsconfig.json**: TypeScript configuration

### Font Issues and Workarounds:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mtmarctoni/story-game-ai](https://github.com/mtmarctoni/story-game-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
