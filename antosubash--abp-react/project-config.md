---
trigger: always_on
description: This document provides comprehensive guidelines for Gemini to follow when working with the AbpReact project, ensuring consistent and high-quality development practices.
---

# Gemini Guidelines for AbpReact

This document provides comprehensive guidelines for Gemini to follow when working with the AbpReact project, ensuring consistent and high-quality development practices.

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Development Environment](#development-environment)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Development Workflow](#development-workflow)
- [Testing Strategy](#testing-strategy)
- [Code Quality Standards](#code-quality-standards)
- [Component Development](#component-development)
- [API Integration](#api-integration)
- [Deployment](#deployment)
- [Troubleshooting](#troubleshooting)

## 🎯 Project Overview

AbpReact is a modern, full-stack web application template that combines the power of ASP.NET Core with the flexibility of React. It serves as a drop-in replacement for the default Angular UI in ABP Framework projects, offering:

- **Enhanced Performance**: Optimized with Next.js 15 and modern React patterns
- **Better SEO**: Server-side rendering and static generation capabilities
- **Visual Page Building**: Integrated Puck editor for drag-and-drop page creation
- **Component Documentation**: Comprehensive Storybook integration
- **Modern Development Experience**: TypeScript, Tailwind CSS, and advanced tooling

### Key Features

- 🎨 **Visual Editor**: Puck-based page builder with drag-and-drop functionality
- 📚 **Component Library**: Extensive UI component library with Storybook documentation
- 🔐 **Authentication**: Integrated ABP authentication with OpenID Connect
- 👥 **Multi-tenancy**: Full support for ABP's multi-tenant architecture
- 📱 **Responsive Design**: Mobile-first approach with Tailwind CSS
- 🧪 **Testing**: Comprehensive testing with Vitest and Storybook testing
- 🚀 **Performance**: Optimized bundle size and loading performance

## 🏗️ Architecture

### Frontend Architecture

```
src/
├── app/                    # Next.js app router pages
│   ├── admin/             # Admin interface pages
│   ├── auth/              # Authentication routes
│   └── pages/             # Public pages
├── components/            # Reusable UI components
│   ├── ui/               # Base UI components (Radix UI)
│   ├── puck/             # Puck editor components
│   └── [feature]/        # Feature-specific components
├── client/               # API client and types
├── hooks/                # Custom React hooks
├── lib/                  # Utility functions and configurations
└── stories/              # Storybook stories
```

### Backend Integration

- **ABP Framework**: .NET 8 backend with full ABP features
- **OpenAPI**: Auto-generated TypeScript client from OpenAPI spec
- **Authentication**: ABP's OpenID Connect implementation
- **Multi-tenancy**: Tenant-aware API calls and UI

## 🛠️ Development Environment

### Prerequisites

- **Node.js**: 18.x or higher
- **pnpm**: 8.x or higher (recommended package manager)
- **.NET 8 SDK**: For backend development
- **Git**: Version control

### Tech Stack

| Category | Technology | Version | Purpose |
|----------|------------|---------|---------|
| **Frontend Framework** | Next.js | 15.x | React framework with SSR/SSG |
| **UI Library** | React | 18.x | Component library |
| **Styling** | Tailwind CSS | 4.x | Utility-first CSS framework |
| **UI Primitives** | Radix UI | Latest | Accessible component primitives |
| **State Management** | TanStack Query | Latest | Server state management |
| **Forms** | React Hook Form + Zod | Latest | Form handling and validation |
| **Type Safety** | TypeScript | 5.x | Static type checking |
| **Testing** | Vitest | Latest | Unit and integration testing |
| **Documentation** | Storybook | Latest | Component documentation |
| **Code Quality** | ESLint + Prettier | Latest | Linting and formatting |

## 🚀 Getting Started

### 1. Clone and Setup

```bash
# Clone the repository
git clone <repository-url>
cd AbpReact

# Navigate to frontend directory
cd src

# Install dependencies
pnpm install
```

### 2. Environment Configuration

Create a `.env.local` file in the `src` directory:

```env
# API Configuration
NEXT_PUBLIC_API_URL=http://localhost:44300
NEXT_PUBLIC_APP_NAME=AbpReact

# Authentication
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-secret-key

# Optional: Analytics
NEXT_PUBLIC_GOOGLE_ANALYTICS_ID=your-ga-id
NEXT_PUBLIC_UMAMI_ID=your-umami-id
```

### 3. Start Development

```bash
# Start development server
pnpm dev

# The application will be available at http://localhost:3000
```

## 📁 Project Structure

### Key Directories

```
src/
├── app/                    # Next.js app router
│   ├── admin/             # Admin interface
│   │   ├── cms/          # Content management
│   │   ├── permissions/  # Permission management
│   │   ├── profile/      # User profile
│   │   ├── settings/     # Application settings
│   │   ├── tenants/      # Tenant management
│   │   └── users/        # User management
│   ├── auth/             # Authentication routes
│   └── pages/            # Public pages
├── components/            # Reusable components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antosubash/abp-react](https://github.com/antosubash/abp-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
