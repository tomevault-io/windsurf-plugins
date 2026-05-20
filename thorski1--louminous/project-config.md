---
trigger: always_on
description: - framework: "Next.js@15.0.0"
---

# LMS MVP Project Rules
# This file defines the rules and processes for our Learning Management System project

# Technology Stack Requirements
technologies:
  frontend:
    - framework: "Next.js@15.0.0"
    - styling: "tailwindcss@latest"
    - ui:
      - "shadcn-ui@latest"
      - "aceternity-ui@latest"
      - "magic-ui@latest"
      - "vaul@latest"
      - "sonner@latest"
      - "cmdk@latest"
    - state: "nuqs@latest"
    - forms: "@hookform/resolvers@latest"
    - validation: "zod@latest"
    - animation:
      - "framer-motion@latest"
      - "@legendapp/motion@latest"
      - "@formkit/auto-animate@latest"
  
  backend:
    - cms: "payload@3.0.0"
    - orm: "drizzle-orm@latest"
    - database: "neon@latest"
    - auth:
        current: "payload-auth@3.0.0"
        future: "clerk@latest"
    - ai: "@vercel/ai@latest"
    - payments: "stripe@latest"
    - uploads: "uploadthing@latest"
    - email: "resend@latest"
    - realtime: "pusher@latest"

  deployment:
    - platform: "vercel@latest"
    - monitoring:
      - "sentry@latest"
      - "@google-cloud/logging@latest"
      - "@google-cloud/opentelemetry-cloud-monitoring-exporter@latest"
      - "@google-cloud/opentelemetry-cloud-trace-exporter@latest"
    - analytics: "vercel-analytics@latest"

  compatibility:
    notes:
      - "Payload CMS 3.0 requires Next.js 15 for optimal performance"
      - "Server Actions are fully supported in Next.js 15"
      - "App Router is the recommended approach"
      - "React Server Components are fully utilized"
      - "nuqs provides URL-based state management compatible with RSC"

# State Management Guidelines
state:
  principles:
    - "Prefer server-side state management over client-side"
    - "Use URL state for shareable and bookmarkable UI states"
    - "Leverage React Server Components for data fetching"
    - "Minimize client-side JavaScript"

  urlState:
    usage:
      - "Search parameters and filters"
      - "Pagination state"
      - "Tab selections"
      - "Modal states"
      - "Sort orders"
      - "View preferences"
    
    implementation:
      - "Use nuqs for type-safe URL state management"
      - "Define searchParams types with Zod schemas"
      - "Implement default values for all URL parameters"
      - "Handle URL parameter validation"
    
    benefits:
      - "SEO-friendly state management"
      - "Shareable URLs with complete state"
      - "Reduced client-side JavaScript"
      - "Better caching capabilities"
      - "Native browser history support"

  serverState:
    patterns:
      - "Use Server Components for data fetching"
      - "Implement server actions for mutations"
      - "Cache responses appropriately"
      - "Handle loading and error states"

  clientState:
    limitations:
      - "Restrict to ephemeral UI state only"
      - "Use React.useState for temporary form state"
      - "Avoid redundant client-side state"
      - "Prefer URL state when possible"

# Collections Structure
collections:
  core:
    - Tenants
    - Users
    - Courses
    - Modules
    - Lessons
    - Quizzes
    - Assignments
    - Submissions

  gamification:
    - Points
    - Badges
    - Achievements
    - Streaks
    - Leaderboard

  communication:
    - Notifications
    - Collaborations
    - Announcements
    - SupportTickets

  settings:
    - TenantSettings
    - StudentSettings
    - LearningPaths

# File Structure
structure:
  app:
    - "(auth): Authentication routes"
    - "(dashboard): Protected dashboard routes"
    - "api: API routes when necessary"
    - "public: Static assets"
    
  components:
    - "ui: Reusable UI components"
    - "forms: Form components"
    - "layouts: Layout components"
    
  lib:
    - "db: Database configuration"
    - "utils: Utility functions"
    - "validation: Schema validation"
    
  types:
    - "Global type definitions"

# Code Standards
standards:
  general:
    - "Use TypeScript strict mode"
    - "Implement proper error handling"
    - "Add comprehensive logging"
    - "Document all public APIs"
    - "Follow SOLID principles"

  nextjs:
    - "Use server components by default"
    - "Implement proper data fetching patterns"
    - "Optimize for performance"
    - "Follow app router best practices"

  database:
    - "Use Drizzle migrations"
    - "Implement proper indexing"
    - "Handle database errors gracefully"
    - "Use transactions where needed"

  security:
    - "Implement proper authentication"
    - "Use role-based access control"
    - "Sanitize all inputs"
    - "Protect sensitive data"

  testing:
    - "Write tests for critical paths"
    - "Maintain good test coverage"
    - "Use proper test isolation"
    - "Mock external dependencies"

# Multi-Agent Development Process
agents:
  architect:
    role: "System Design & Architecture"
    responsibilities:
      - "Review and approve architectural decisions"
      - "Ensure scalability and performance"
      - "Maintain system consistency"
      - "Plan data structures and relationships"
    triggers:
      - "New feature proposal"
      - "Architecture changes"
      - "Performance optimization requests"
      - "Database schema updates"

  developer:
    role: "Code Implementation"
    responsibilities:
      - "Write clean, maintainable code"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thorski1/LouMinouS](https://github.com/thorski1/LouMinouS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
