---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Never mention claude code in git commit messages.
Never try to create git commit without me asking so.

## Project Overview

**Auteurium** is a web application for creating text snippets and organizing them visually on a canvas with logical connections. The application is built with a complete AWS serverless architecture and React frontend, ready for development and testing.

### Core Concept
- Users create text snippets within projects
- Snippets are positioned on a visual canvas (similar to Miro/FigJam)
- Snippets can be connected with directional, many-to-many relationships
- Each snippet has a primary text field (textField1) and a title field, plus tags/categories
- Inline editing of title and text directly on canvas nodes (click to edit, Cmd/Ctrl+Enter to save)
- Canvas supports zooming, panning, and drag-and-drop positioning

## Architecture

**Monorepo Structure**: All components (frontend, backend, infrastructure) will be organized in a single GitHub repository

**Technology Stack**:
- **Frontend**: React with React Flow for canvas interactions, Tailwind CSS for styling, AWS Amplify v6
- **Backend**: AWS Lambda (Node.js with TypeScript), GraphQL resolvers
- **API Layer**: GraphQL with AWS AppSync
- **Authentication**: AWS Cognito with JWT validation
- **Storage**: DynamoDB with Global Secondary Indexes (designed for Neptune migration)
- **Infrastructure**: AWS CDK for Infrastructure as Code
- **Hosting**: Amazon S3 + CloudFront (disabled for development security)

## Key Application Features

### User Management
- Self-registration with email/password
- Admin users with system analytics access (cannot view snippet content)
- All user data is private and scoped to individual users

### Project Organization
- Users can create multiple projects
- Snippets, tags, and categories are project-specific
- Project deletion cascades to all contained snippets

### Snippet Management
- Each snippet has a title field and primary text field (textField1), no length limits (expected up to page length)
- Unique alphanumeric IDs displayed on canvas (shortened to 8 characters)
- Large snippets (>100 words) shown minimized with expand modal
- Version history with revert capability
- Positioned freely on infinite canvas
- **Inline Editing**: Click title or text field directly on canvas node to edit, Cmd/Ctrl+Enter to save, Esc to cancel
- **Connected Content Aggregation**: Snippets automatically aggregate content from upstream connections
- **Prompt Designer**: Interactive prompt builder for AI generation with connected content preview
- **AI-Powered Image Generation**: Multiple models (Google Imagen 4 Fast, Gemini 2.5 Flash Image with multimodal support for up to 3 connected images)
- **Text-to-Text Generation**: LLM-powered snippet creation with model selection
- **Model Selection**: Choose text, image, or video models directly on snippet nodes
- **Optimistic Updates**: Instant UI feedback with background synchronization

### Connection System
- Directional relationships between snippets (A depends on B)
- Many-to-many connections supported
- Connection labels/tags for relationship types
- Connections created by entering target snippet ID
- Visual lines drawn between connected snippets

### Canvas Interface
- Infinite scrollable workspace per project
- Pan and zoom controls
- Drag-and-drop snippet positioning
- Context menus for snippet operations
- Modal dialogs for editing and connection management

## Development Commands

**Prerequisites**: Node.js 22+ (LTS), npm 10+, TypeScript 5.9+, AWS CLI configured (for deployment)

### Setup
- `npm install` - Install all workspace dependencies

### Development
- `npm run dev` - Start web app (localhost:3000) and API service concurrently
- `npm run dev:web` - Start only web app with Vite dev server
- `npm run dev:api` - Start only API service in watch mode

### Building
- `npm run build` - Build entire monorepo using automated script
- `cd apps/web && npm run build` - Build React web application
- `cd services/api && npm run build` - Build API Lambda functions
- `cd infrastructure/aws-cdk && npm run build` - Build CDK infrastructure
- `npm run generate` - Generate GraphQL types and schemas
- Build dependencies: shared-types → validation → graphql-schema → api → web → infrastructure

**Build Scripts**:
- `tools/scripts/build-all.sh` - Automated build process for all components
- `tools/scripts/generate-graphql.sh` - GraphQL schema and type generation
- `tools/scripts/setup-dev.sh` - Development environment setup
- `tools/scripts/deploy-stack.sh` - Deployment automation

### Code Quality
- `npm run lint` - Run linting for all workspaces (web + API build check)
- `npm run typecheck` - Run TypeScript type checking for all workspaces
- `cd apps/web && npm run lint` - ESLint for React frontend only
- `cd apps/web && npm run typecheck` - TypeScript type checking for frontend only

### Testing
- `cd services/api && npm run test` - Run Jest unit tests for API resolvers
- `npm run test` - Run all tests (API + integration tests from root)
- `npm run test:e2e` - Run Playwright end-to-end tests across all browsers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [twokey/auteurium-prototype](https://github.com/twokey/auteurium-prototype) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
