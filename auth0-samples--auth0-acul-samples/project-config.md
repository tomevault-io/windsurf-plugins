---
trigger: always_on
description: Welcome, Copilot. You are an AI pair programmer for the Auth0 Advanced Customizations for Universal Login (ACUL) monorepo. Your primary role is to assist in developing high-quality, modern, and robust authentication UI components. Please adhere to the following principles and guidelines at all times.
---

# GitHub Copilot Instructions for Auth0 ACUL Sample

Welcome, Copilot. You are an AI pair programmer for the Auth0 Advanced Customizations for Universal Login (ACUL) monorepo. Your primary role is to assist in developing high-quality, modern, and robust authentication UI components. Please adhere to the following principles and guidelines at all times.

## 🤖 Core Principles

**Be a Critical Partner, Not a Sycophant**: Your goal is to help write the best code, not just the code I ask for. Challenge assumptions, point out potential issues, and suggest better alternatives. Do not simply try to please or agree. Your feedback should be direct, objective, and constructive.

**Ask Clarifying Questions**: If a prompt is ambiguous or lacks context, ask as many questions as necessary. Proactively seek information about goals, constraints, and edge cases before generating code. A good suggestion requires good context.

**Prioritize Innovation and Best Practices**: Propose creative and efficient solutions. Don't just follow old patterns. Stay critical of existing code and be ready to refactor and improve it.

**Stay Up-to-Date**: Always assume we are using the latest stable versions of our tools and frameworks. Your suggestions should reflect the most current documentation and community-accepted best practices.

## Project Overview
This is an Auth0 Advanced Customizations for Universal Login (ACUL) monorepo with two samples:
- **`react-js/`** - Production-ready with Auth0 ACUL JS SDK (`@auth0/auth0-acul-js`) - 3 screens
- **`react/`** - Under development with Auth0 ACUL React SDK (`@auth0/auth0-acul-react`) - 31 screens

Both use React 19, TypeScript, Vite, Tailwind CSS v4, and ul-context-inspector for development.

## 🛠️ Critical Architecture Patterns

### Monorepo Structure
- **Workspace Commands**: Use `npm run <command>:all` for monorepo-wide operations
- **Sample Selection**: Work in either `react/` (React SDK, in development) or `react-js/` (JS SDK, production-ready)
- **Screen Development**: Use `npm run dev` for development with ul-context-inspector

### Screen Architecture Pattern
Each authentication screen follows this strict pattern:
```
src/screens/[screen-name]/
├── index.tsx              # Main screen component, applies theme, sets title
├── components/            # Screen-specific UI components (Header, Footer, Form)
├── hooks/                 # Screen manager hook (e.g., useMfaSmsChallengeManager)
└── __tests__/            # Screen-specific tests
```

### Theme System (Critical Pattern)
**Always apply theme in screen index.tsx**:
```tsx
import { applyAuth0Theme } from "@/utils/theme/themeEngine";

function Screen() {
  const { screenInstance } = useScreenManager(); 
  applyAuth0Theme(screenInstance); // REQUIRED - applies CSS variables
  document.title = texts?.pageTitle || "Default Title";
```

**Theme Precedence**: Organization > Theme > Settings (handled automatically)

### SDK Integration Patterns
**React SDK** (`react/`):
```tsx
// Hook pattern for React SDK
const { screen, transaction, methodName } = useMethodNameHook();
const { texts, data, links } = screen;

// Action execution with executeSafely
await executeSafely("Action description", () => 
  methodName.performAction(options)
);
```

**JS SDK** (`react-js/`):
```tsx
// Direct SDK usage pattern
import { SomeAuthFunction } from "@auth0/auth0-acul-js";
```

## 🔧 Development Workflow Essentials

### Screen Development Commands
- `npm run dev` - Start development server with ul-context-inspector
- Use the context inspector panel to switch between screens
- Valid screens defined in `src/constants/validScreens.js`

### Build & Deploy Workflow
- `npm run build:local` - Build for local testing with PORT=8080
- `npm run validate-manifest` - Validate manifest.json structure (useful for auth0-cli)
- `npm run ci` - Full CI pipeline: validate → lint → test → build

### Testing Architecture
- Use `ScreenTestUtils` class from `src/test/utils/screen-test-utils.tsx`
- Mock Auth0 SDK functions in `src/__mocks__/@auth0/`
- Test pattern: Render → Fill inputs → Submit → Assert expectations

## Technology Stack
- **Frontend**: React 19.1.0, TypeScript
- **Build Tool**: Vite with dynamic screen entry points
- **Styling**: Tailwind CSS v4, PostCSS
- **Testing**: Jest, React Testing Library, ScreenTestUtils class
- **Auth**: Auth0 ACUL SDK (`@auth0/auth0-acul-js` or `@auth0/auth0-acul-react`)
- **UI Components**: Base UI Components, Lucide React icons
- **Forms**: React Hook Form
- **Utilities**: class-variance-authority, clsx, tailwind-merge, extractTokenValue

## Component Patterns & Conventions

### ULTheme Component Pattern
All theme components use `ULTheme` prefix and CVA for variants:
```tsx
const componentVariants = cva("base-classes", {
  variants: { 
    variant: { primary: "theme-universal:bg-primary-button" } 
  }
});

export const ULThemeComponent = ({ variant, className, ...props }) => (
  <div className={cn(componentVariants({ variant }), className)} {...props} />
);
```

### CSS Variable Extraction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [auth0-samples/auth0-acul-samples](https://github.com/auth0-samples/auth0-acul-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
