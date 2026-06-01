---
trigger: always_on
description: EasyTransfer is a free, anonymous, encrypted, and easy-to-use E2EE file transfer tool built with WebRTC and Vue.js. The project enables peer-to-peer file transfers between devices across any network using simple device codes.
---

# GitHub Copilot Instructions for EasyTransfer

## Project Overview

EasyTransfer is a free, anonymous, encrypted, and easy-to-use E2EE file transfer tool built with WebRTC and Vue.js. The project enables peer-to-peer file transfers between devices across any network using simple device codes.

## Project Structure

```
/client       - Vue 3 + TypeScript frontend application
/server       - Node.js + TypeScript backend server (Socket.io)
/assets       - Static assets (images, icons)
/.github      - GitHub workflows and templates
```

## Technology Stack

### Client

- **Framework**: Vue 3 with Composition API
- **Language**: TypeScript
- **Build Tool**: Vite
- **State Management**: Pinia
- **Real-time Communication**: Socket.io-client, WebRTC
- **Styling**: Sass/SCSS
- **Code Quality**: ESLint, Prettier

### Server

- **Runtime**: Node.js
- **Language**: TypeScript
- **Framework**: Socket.io
- **Build**: TypeScript Compiler (tsc)
- **Code Quality**: ESLint, Prettier

## Development Commands

### Client (in `/client` directory)

```bash
npm run dev         # Start development server
npm run build       # Build for production
npm run preview     # Preview production build
npm run lint        # Run ESLint with auto-fix
npm run format      # Format code with Prettier
npm test            # Run tests
npm run test:watch  # Run tests in watch mode
npm run test:ui     # Run tests with UI
```

### Server (in `/server` directory)

```bash
npm run dev         # Build and start with nodemon
npm run build       # Compile TypeScript
npm run start       # Start production server
npm run lint        # Run ESLint with auto-fix
npm run format      # Format code with Prettier
npm test            # Run tests
npm run test:watch  # Run tests in watch mode
npm run test:ui     # Run tests with UI
```

## Commit Message Guidelines

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification for semantic commit messages:

### Format

```plaintext
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types

- **feat**: A new feature
- **fix**: A bug fix
- **docs**: Documentation only changes
- **style**: Changes that don't affect code meaning (formatting, whitespace)
- **refactor**: Code change that neither fixes a bug nor adds a feature
- **perf**: Performance improvement
- **test**: Adding or updating tests
- **build**: Changes to build system or dependencies
- **ci**: Changes to CI configuration files and scripts
- **chore**: Other changes that don't modify src or test files

### Examples

```
feat(client): add dark mode theme support
fix(server): resolve connection timeout issue
docs(readme): update installation instructions
test(client): add FileChunkManager unit tests
refactor(utils): extract ID generation to utils
ci: add automated testing to GitHub Actions
chore(dependencies): update dependencies to latest versions
```

### Best Practices

- Use present tense (`add` not `added`)
- Use imperative mood (`move` not `moves`)
- Keep first line under 72 characters
- Reference issues/PRs in footer: `Fixes #123`
- Break long descriptions into body paragraphs

## Code Style Guidelines

### General

- Use TypeScript for type safety
- Follow existing code patterns and conventions
- Write clear, self-documenting code
- Keep functions small and focused
- Use meaningful variable and function names

### Vue Components

- Use Composition API with `<script setup>` syntax
- Define props with TypeScript interfaces
- Use reactive state management with Pinia stores
- Keep components focused on single responsibilities
- Use Vue 3 best practices (ref, reactive, computed, watch)

### TypeScript

- Enable strict type checking
- Avoid `any` type - use proper types or `unknown`
- Define interfaces for complex data structures
- Use type guards for runtime type checking
- Export types that are used across files

### File Naming

- Vue components: PascalCase (e.g., `FileTransfer.vue`)
- TypeScript files: camelCase (e.g., `socketManager.ts`)
- Utilities: camelCase with descriptive names
- Constants: UPPER_SNAKE_CASE or camelCase based on context

### Code Organization

- Keep related functionality together
- Separate business logic from UI components
- Use utility functions for shared logic
- Store configuration in dedicated files
- Define types in separate type files when shared

## Key Features to Understand

### WebRTC Implementation

- Peer-to-peer connection establishment
- STUN/TURN server configuration
- Data channel management
- ICE candidate handling

### Socket.io Integration

- Device code generation and management
- Signaling server for WebRTC
- Connection state management
- Error handling and reconnection logic

### File Transfer Flow

1. Device code generation
2. Peer connection via code
3. WebRTC connection establishment
4. File chunking and transfer
5. Progress tracking
6. Transfer completion/error handling

## Testing Considerations

The project uses Vitest for unit testing:

- **Client**: 157 tests covering utilities, protocol handling, retry logic, theming, and WebRTC

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WCY-dt/EasyTransfer](https://github.com/WCY-dt/EasyTransfer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
