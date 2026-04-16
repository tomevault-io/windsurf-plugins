---
trigger: always_on
description: You are working on **SilentSort** (previously FlowGenius), an AI-powered desktop application that automatically organizes and renames files using content analysis. This is a 4-day intensive development project focused on solving personal productivity problems through intelligent automation.
---

# SilentSort - AI-Powered File Organization Desktop App
# Cursor Rules & Coding Standards

## PROJECT CONTEXT
You are working on **SilentSort** (previously FlowGenius), an AI-powered desktop application that automatically organizes and renames files using content analysis. This is a 4-day intensive development project focused on solving personal productivity problems through intelligent automation.

### CORE PRODUCT
- **Purpose**: AI watches files, reads content, automatically renames & organizes them
- **Platform**: macOS desktop application (Electron-based)
- **Key Features**: Auto-rename, auto-organize, smart search, background processing
- **User Experience**: Silent background operation with system tray interface

## DOMAIN EXPERTISE AREAS

### 1. DESKTOP APPLICATION DEVELOPMENT
- Electron framework with React/TypeScript frontend
- macOS system integration (File System Events API, LaunchAgent, code signing)
- System tray application patterns and UX best practices
- Background process management and resource optimization
- Cross-platform desktop development considerations

### 2. AI/ML INTEGRATION & WORKFLOW AUTOMATION  
- OpenAI API integration for content analysis and file naming
- LangGraph for AI workflow orchestration and state management
- LangSmith for AI monitoring and debugging
- n8n Cloud for workflow automation (no local Docker complexity)
- Content analysis algorithms for various file types (PDF, images, documents)

### 3. FILE SYSTEM MANAGEMENT
- Real-time file monitoring using chokidar and File System Events
- Safe file operations (rename, move, organize) with rollback capabilities
- macOS file permissions and security considerations
- Spotlight integration for enhanced search capabilities
- File type detection and content extraction patterns

### 4. DATABASE & CLOUD ARCHITECTURE
- Hybrid Supabase (cloud) + SQLite (local) architecture
- Data synchronization between local and cloud storage
- File metadata storage and indexing strategies
- User settings and preferences management
- Offline-first design with cloud backup capabilities

### 5. USER EXPERIENCE & INTERFACE DESIGN
- Minimal intrusion UI patterns for productivity tools
- System tray integration and menu design
- File organization decision workflows (approve/deny AI suggestions)
- Global hotkey implementation (Cmd+Shift+F for search)
- Progressive disclosure UI principles

## TECHNICAL STACK & FRAMEWORKS

### Core Technologies
```
Frontend: Electron + React 18+ + TypeScript
Backend: Node.js 18+ + Supabase + SQLite
AI Services: OpenAI API + LangGraph + LangSmith
Automation: n8n Cloud workflows
Platform: macOS 12.0+ (Universal binary)
```

### Key Libraries & Dependencies
- **chokidar** - File system monitoring
- **electron-builder** - Application packaging and distribution
- **sqlite3** - Local database operations
- **@supabase/supabase-js** - Cloud database integration
- **openai** - AI content analysis
- **langchain** - AI workflow management
- **electron-store** - Settings persistence

## CODING STANDARDS & BEST PRACTICES

### TypeScript Configuration
- Use strict mode with all strict checks enabled
- Prefer explicit typing over `any`
- Use proper interface definitions for all data structures
- Implement proper error types and handling
- Use utility types for API responses and file operations
- **NO EMOJIS in code** - Keep all code, comments, variable names, and function names emoji-free

### React Best Practices
- Use functional components with hooks exclusively
- Implement proper error boundaries for AI operations
- Use React.memo for performance optimization of file lists
- Follow container/presentation component patterns
- Implement proper cleanup in useEffect for file watchers

### Electron Security & Performance
- Enable context isolation and disable node integration in renderers
- Use proper IPC communication patterns between main and renderer
- Implement CSP (Content Security Policy) headers
- Handle file system permissions properly
- Optimize memory usage for large file operations



### Build & Deployment
- Use electron-builder for creating distributable packages
- Implement proper code signing for macOS distribution
- Create both development and production build configurations
- Include auto-updater functionality for seamless updates

## AI-SPECIFIC GUIDELINES

### Content Analysis Best Practices
- Always include confidence scores with AI predictions
- Implement fallback strategies when AI services are unavailable
- Cache AI results to avoid redundant API calls
- Respect API rate limits with proper queuing
- Provide user override capabilities for all AI decisions

### Workflow Orchestration
- Use LangGraph for complex multi-step AI workflows
- Implement proper state management for long-running processes
- Handle partial failures gracefully in workflow chains
- Monitor AI performance and accuracy over time
- Allow users to customize AI behavior through settings

## SECURITY & PRIVACY CONSIDERATIONS

### Data Handling
- Process files locally when possible to protect privacy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pranjalekhande) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
