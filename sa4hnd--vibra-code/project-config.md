---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Development Standards

### Recommended MCP Tools & Skills

**Use these tools when implementing iOS features:**

| Tool | Purpose | When to Use |
|------|---------|-------------|
| **Apple Docs MCP** | Official Apple documentation lookup | API signatures, framework features, deprecations |
| **Apple HIG Skill** | Human Interface Guidelines compliance | UI design, interactions, accessibility |
| **Context7 MCP** | Library documentation | Third-party frameworks (IGListKit, Texture, etc.) |
| **Exa Search** | Code examples, debugging solutions | Stack Overflow, GitHub issues, forum answers |

**Workflow:**
1. **Before coding**: Check Apple docs for APIs
2. **During coding**: Verify HIG compliance, read Apple docs for best practices
3. **After coding**: Run audits (accessibility, performance, concurrency)

### Quality Standards

- **60fps minimum** - No frame drops, jank, or stuttering
- **HIG compliant** - Follow Apple Human Interface Guidelines strictly
- **iOS 26+ optimized** - Use newest Apple APIs and approaches
- **Battery efficient** - Anti-battery-drainage patterns mandatory
- **Thermal aware** - Minimize CPU/GPU heat generation
- **Accessibility first** - VoiceOver, Dynamic Type, Reduce Motion support

### Performance Requirements

**Every implementation must be optimal:**

```
BAD: "It works, ship it"
GOOD: "It works, AND it's:
   - Off-main-thread (Texture/GCD)
   - Memory efficient (no leaks, proper cleanup)
   - Battery friendly (no timers/polling when not needed)
   - Following Apple's recommended pattern from WWDC 2025"
```

**Before implementing ANY feature:**
1. Read Apple documentation for the recommended approach
2. Check WWDC sessions for latest best practices
3. Consider battery, memory, and thermal impact
4. Use Instruments to verify performance

### iOS 26 Best Practices

**Always prefer modern approaches:**

| Old Pattern | Modern Pattern (iOS 26+) |
|-------------|--------------------------|
| `UIBlurEffect` | `UIGlassEffect` (Liquid Glass) |
| Manual layout | `ASLayoutSpec` / SwiftUI |
| `Timer` polling | Combine / async-await observers |
| `URLSession` callbacks | `async/await` networking |
| `ObservableObject` | `@Observable` macro |
| Manual animations | Spring animations with `UIView.animate(springDuration:)` |

---

## Project Overview

**Vibra Code** is an AI-powered mobile app builder. Users describe apps in plain English, and AI builds them in cloud sandboxes with native preview on their phones.

### Repository Structure

- `vibracode-mobile/apps/expo-go/` - **Vibra Code mobile app** (modified Expo Go)
- `vibracode-backend/` - **Website and backend** (Next.js + Convex + Inngest)

## Development Commands

### vibracode-backend (Backend/Website)

```bash
cd vibracode-backend

# Start development (Next.js with Turbopack)
npm run dev

# Start full stack (Next.js + Inngest dev server)
npm run start

# Start Inngest background functions only (port 8288)
npx inngest-cli@latest dev

# Build for production
npm run build

# Lint
npm run lint

# Deploy Convex schema changes
npx convex deploy
```

### vibracode-mobile (Mobile App)

```bash
cd vibracode-mobile/apps/expo-go

# Start Metro bundler (must run on port 80)
yarn start

# Run tests
yarn test

# Lint
yarn lint

# Generate GraphQL types
yarn generate-graphql-code

# iOS build (from ios directory)
cd ios && pod install
# Then open Exponent.xcworkspace in Xcode

# Android build (from android directory)
cd android && ./gradlew app:assembleDebug
```

## Architecture

### Data Flow

1. User describes app in mobile app (VibraCreateAppScreen)
2. Mobile app calls backend API -> creates session in Convex
3. Inngest queues `create-session` function -> spawns E2B sandbox
4. Sandbox clones Expo template, installs deps, starts dev server
5. Inngest runs `run-agent` function -> AI generates code
6. Real-time updates stream via Convex to mobile app
7. Mobile app opens native preview via tunnel URL

### Backend Stack

- **Next.js 15** - App Router, Server Actions, API routes
- **Convex** - Real-time database, sessions/messages/users schema
- **Inngest** - Background job processing (create-session, run-agent, push-to-github)
- **E2B** - Cloud sandboxes for code execution
- **Clerk** - Authentication
- **Stripe** - Web payments (optional)
- **Claude Agent SDK** - AI code generation

### Mobile Stack

- **React Native / Expo SDK 54** - Native mobile runtime
- **Clerk** - Authentication (shared with backend)
- **Convex** - Real-time data sync (symlinked to vibracode-backend/convex)
- **RevenueCat** - In-app purchases (optional)
- **React Navigation** - Stack and bottom tab navigation

### Key Directories

**vibracode-backend:**
- `app/api/` - API routes (create-session, run-agent, webhooks)
- `convex/` - Database schema and functions (sessions, messages, billing, usage)
- `lib/inngest/functions/` - Background jobs (create-session.ts, run-agent.ts, push-to-github.ts)
- `lib/prompts.ts` - AI system prompts
- `lib/e2b/` - E2B sandbox configuration
- `lib/revenuecat/` - RevenueCat webhook handlers
- `e2b-cursor-template/` - E2B sandbox Dockerfile

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sa4hnd/vibra-code](https://github.com/sa4hnd/vibra-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
