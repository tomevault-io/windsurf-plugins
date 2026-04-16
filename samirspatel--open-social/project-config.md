---
trigger: always_on
description: GitSocial is a distributed social media application where users' data lives in their own GitHub repositories, implementing true data ownership and portability. Each user controls their social graph through structured JSON files in their personal GitHub repo.
---

# GitSocial Development Plan & Implementation Guide

## Project Overview
GitSocial is a distributed social media application where users' data lives in their own GitHub repositories, implementing true data ownership and portability. Each user controls their social graph through structured JSON files in their personal GitHub repo.

## Core Technology Stack
- **Frontend**: Next.js 14, React 18, TypeScript, Tailwind CSS
- **Backend**: Node.js, Express, TypeScript, Prisma
- **Database**: PostgreSQL (cache only), Redis (sessions/realtime)
- **External**: GitHub API, GitHub OAuth, GitHub Webhooks
- **Real-time**: Socket.io
- **Deployment**: Vercel (frontend), Railway/Render (backend)

## Development Phases

### PHASE 1: Foundation & Core Infrastructure (Weeks 1-3)

#### Week 1: Project Setup & GitHub Integration
**Milestone 1.1: Development Environment**
- [ ] Initialize Next.js project with TypeScript
- [ ] Set up ESLint, Prettier, Tailwind CSS
- [ ] Configure environment variables and secrets
- [ ] Set up GitHub repository and basic CI/CD
- [ ] Create basic project structure and documentation

**Milestone 1.2: GitHub API Integration**
- [ ] Set up GitHub OAuth application
- [ ] Implement Octokit client wrapper
- [ ] Create GitHub authentication middleware
- [ ] Build repository management utilities
- [ ] Test basic GitHub API operations (create repo, read/write files)

**Week 2: Data Layer & Schemas**
**Milestone 1.3: Data Schemas & Validation**
- [ ] Define TypeScript interfaces for all entities
- [ ] Create JSON schemas for validation (Zod)
- [ ] Build data access layer with repository pattern
- [ ] Implement cryptographic signing utilities
- [ ] Create repository initialization logic

**Milestone 1.4: Core Data Operations**
- [ ] Profile CRUD operations
- [ ] Post creation and retrieval
- [ ] Basic social connections (follow/unfollow)
- [ ] File-based storage operations with Git commits
- [ ] Error handling and validation

**Week 3: Basic Web Interface**
**Milestone 1.5: Authentication & User Onboarding**
- [ ] GitHub OAuth login flow
- [ ] User session management
- [ ] Repository initialization on first login
- [ ] Basic profile setup UI
- [ ] Error handling and user feedback

**Milestone 1.6: Core UI Components**
- [ ] Layout and navigation components
- [ ] Profile display and editing
- [ ] Basic post creation form
- [ ] Simple feed display
- [ ] Responsive design foundation

### PHASE 2: Social Features & Feed System (Weeks 4-6)

#### Week 4: Social Graph Management
**Milestone 2.1: Following System**
- [ ] Follow/unfollow users functionality
- [ ] Following/followers list management
- [ ] User discovery and search
- [ ] Social graph validation and integrity
- [ ] Mutual follow detection

**Milestone 2.2: Feed Foundation**
- [ ] Basic feed aggregation logic
- [ ] Repository monitoring system
- [ ] Simple chronological feed generation
- [ ] Feed caching strategy
- [ ] Performance optimization for multiple repos

#### Week 5: Content Interaction
**Milestone 2.3: Post Interactions**
- [ ] Like/unlike posts functionality
- [ ] Reply to posts system
- [ ] Repost/share functionality
- [ ] Mention system (@username)
- [ ] Hashtag support (#topic)

**Milestone 2.4: Enhanced UI**
- [ ] Interactive post components
- [ ] Real-time like/reply updates
- [ ] Feed infinite scrolling
- [ ] Post creation with rich text
- [ ] User profile pages with post history

#### Week 6: Real-time Features
**Milestone 2.5: Live Updates**
- [ ] GitHub webhook setup and handling
- [ ] Socket.io integration for real-time UI
- [ ] Live feed updates
- [ ] Real-time notifications
- [ ] Connection status indicators

**Milestone 2.6: Performance & Caching**
- [ ] Feed caching with Redis
- [ ] Database indexing strategy
- [ ] API response optimization
- [ ] Image/media optimization
- [ ] Rate limiting implementation

### PHASE 3: Advanced Features & Polish (Weeks 7-9)

#### Week 7: Media & Rich Content
**Milestone 3.1: Media Upload System**
- [ ] Image upload to GitHub repositories
- [ ] Video/media file handling
- [ ] Media compression and optimization
- [ ] Rich media preview in posts
- [ ] File size and type validation

**Milestone 3.2: Advanced Post Types**
- [ ] Thread/multi-post support
- [ ] Poll creation and voting
- [ ] Link preview generation
- [ ] Code snippet sharing with syntax highlighting
- [ ] Markdown support in posts

#### Week 8: Search & Discovery
**Milestone 3.3: Search System**
- [ ] Full-text search across posts
- [ ] User search and discovery
- [ ] Hashtag trending system
- [ ] Search result ranking
- [ ] Search performance optimization

**Milestone 3.4: Advanced Social Features**
- [ ] User blocking and reporting
- [ ] Content moderation tools
- [ ] Privacy controls per post
- [ ] Lists/groups functionality
- [ ] Direct messaging system

#### Week 9: Mobile & Progressive Web App
**Milestone 3.5: Mobile Experience**
- [ ] Progressive Web App (PWA) setup
- [ ] Mobile-optimized UI components
- [ ] Offline functionality
- [ ] Push notifications
- [ ] Touch gestures and mobile interactions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/samirspatel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
