---
trigger: always_on
description: **Ngobrolin Web Topic Picker** is a static web application designed for selecting podcast discussion topics. The application provides an interactive interface for browsing and randomly selecting topics from the Ngobrolin podcast's GitHub Discussions repository.
---

# Ngobrolin Web Topic Picker - Agent Documentation

## Project Overview

**Ngobrolin Web Topic Picker** is a static web application designed for selecting podcast discussion topics. The application provides an interactive interface for browsing and randomly selecting topics from the Ngobrolin podcast's GitHub Discussions repository.

### Key Features

- **Random Topic Picker**: Click-to-select functionality with visual animation
- **Spin the Wheel**: Interactive wheel interface for fun topic selection
- **Topic Browsing**: Grid view of all topics with sorting capabilities
- **Statistics Dashboard**: Real-time stats showing total topics, votes, comments, and contributors
- **GitHub Sync**: Optional sync button to fetch live data from GitHub Discussions
- **Topic History Tracking**: Remembers picked topics across sessions using localStorage
- **Smart Skip Logic**: Automatically excludes previously picked topics from random selection
- **Reset History**: Clear pick history to start fresh
- **Responsive Design**: Mobile-friendly interface using Tailwind CSS

## Technology Stack

### Core Technologies

- **Astro 5.16.15**: Modern static site generator with component-based architecture
- **Tailwind CSS 4.1.18**: Utility-first CSS framework for rapid UI development
  - Uses Vite plugin integration (`@tailwindcss/vite`)
- **TypeScript**: Strict mode enabled for type safety
- **Vite**: Build tool and development server (included with Astro)

### Supporting Libraries

- **@astrojs/node 9.5.2**: Node.js integration for API routes
- **cheerio 1.2.0**: HTML parsing for GitHub Discussions scraping

## Architecture

### Static-First Design Philosophy

The application is built with a static-first approach:

1. **Static Data Source**: Topics are stored in `/src/data/topics.ts` as a TypeScript array
2. **Client-Side Rendering**: All interactivity happens in the browser
3. **Optional API**: An API route exists for live GitHub sync but is not required
4. **Build-Time Optimization**: Can be fully pre-rendered as static HTML

### Data Flow

```
Static Topics (src/data/topics.ts)
    ↓
Browser Load (Client-Side)
    ↓
Display & Interact (index.astro)
    ↓
[Optional] GitHub Sync Button
    ↓
API Route (src/pages/api/topics.ts) [Optional]
    ↓
Update Client State
```

### Key Architectural Principles

1. **Keep It Static**: No dynamic fetching on page load - everything starts from static data
2. **Client-Side Only**: All interactivity, sorting, and selection happens in the browser
3. **Optional Enhancement**: GitHub sync is user-initiated, not automatic
4. **Type Safety**: TypeScript strict mode catches errors at build time
5. **Component-Based**: Reusable Astro components for maintainability

## File Structure

```
ngobrolin-web-topic-picker/
├── public/                      # Static assets (if any)
├── src/
│   ├── components/
│   │   └── TopicCard.astro     # Reusable topic card component
│   ├── data/
│   │   └── topics.ts           # Static topics data (main data source)
│   ├── layouts/
│   │   └── Layout.astro        # Main layout wrapper
│   ├── lib/
│   │   └── topicHistory.ts     # localStorage utility for pick history
│   ├── pages/
│   │   ├── api/
│   │   │   └── topics.ts       # Optional API route for GitHub sync
│   │   └── index.astro         # Main application page
│   └── types/
│       └── topic.ts            # TypeScript type definitions
├── astro.config.mjs            # Astro configuration
├── package.json                # Dependencies and scripts
├── tailwind.config.js          # Tailwind CSS configuration
├── tsconfig.json               # TypeScript configuration (strict mode)
└── AGENTS.md                   # This file
```

### Key Files Explained

#### `/src/data/topics.ts`

- **Purpose**: Single source of truth for all topics
- **Format**: TypeScript array of `Topic` objects
- **Import**: Used directly in `index.astro` for initial render
- **Update**: Manually maintained or updated via GitHub sync

#### `/src/lib/topicHistory.ts`

- **Purpose**: localStorage utility module for pick history
- **Functions**: getPickedTopicIds(), addPickedTopic(), isTopicPicked(), clearHistory()
- **Usage**: Imported by index.astro for history management

#### `/src/pages/index.astro`

- **Purpose**: Main application interface
- **Contains**: HTML structure, client-side JavaScript, and styling
- **Features**:
  - Topic picker with animation
  - Wheel spinner with canvas rendering
  - Statistics dashboard
  - Sorting controls
  - GitHub sync button
  - History tracking and reset

#### `/src/pages/api/topics.ts`

- **Purpose**: Optional API route for live GitHub data
- **Usage**: Called only when user clicks "Sync from GitHub" button
- **Function**: Scrapes GitHub Discussions and returns updated topics

#### `/src/types/topic.ts`

- **Purpose**: TypeScript interface definition
- **Interface**: `Topic` with properties (id, title, url, author, votes, comments, category)

## Development Setup

### Prerequisites

- Node.js 18+ (recommended)
- npm or yarn package manager

### Installation

```bash
# Install dependencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ngobrolin/pictopic](https://github.com/ngobrolin/pictopic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
