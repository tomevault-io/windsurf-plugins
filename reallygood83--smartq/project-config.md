---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

```bash
# Development
npm run dev              # Start development server with Turbopack (default port 3000)
npm run build           # Build for production
npm run start           # Start production server
npm run lint            # Run ESLint for code quality

# Firebase deployment
firebase deploy --only database    # Deploy database rules only
firebase deploy                   # Deploy all Firebase config
firebase login                    # Login to Firebase CLI (required once)

# Database rules testing
firebase deploy --only database:rules    # Deploy only security rules

# Git workflow (for commits to https://github.com/reallygood83/smartq)
git add .
git commit -m "descriptive message"
git push origin main
```

## Architecture Overview

**SmartQ** is an AI-powered educational platform built with Next.js 15 and Firebase, designed to help teachers collect and analyze student questions using Google Gemini API.

### Core Architecture Patterns

**Education Level Adaptation System**
- All UI/UX adapts to 5 education levels: Elementary, Middle, High, University, Adult
- `EducationLevelContext` provides level-specific terminology, themes, and AI prompts
- Components use `useSmartTerminology()` for adaptive language
- Theme system in `/src/styles/themes.ts` provides level-specific visual styling

**Dual Authentication Model**
- Teachers: Firebase Authentication (Google OAuth)
- Students: Anonymous access via 6-digit session codes
- Security rules enforce teacher-only data modification

**Client-Side API Key Management**
- User-provided Gemini API keys stored encrypted in localStorage
- Never stored on server - encryption/decryption in `/src/lib/encryption.ts`
- API keys are user-specific and tied to Firebase auth UID

**Real-time Data Architecture**
- Firebase Realtime Database for live updates
- Session-based data structure: `/sessions/{sessionId}/`
- Questions, shared content, and AI analyses linked to sessions
- Students write to `/questions/{sessionId}/`, teachers control everything else

### Key Data Structures

**Sessions**: Core teaching units with unique 6-digit access codes
```typescript
{
  sessionId: string,
  title: string,
  accessCode: string,  // 6-digit student access code
  sessionType: 'DEBATE' | 'INQUIRY' | 'PROBLEM' | 'CREATIVE' | 'DISCUSSION' | 'QNA',
  teacherId: string,
  subjects?: Subject[],
  isAdultEducation?: boolean,
  
  // Teacher-led mode support (NEW)
  interactionMode?: 'free_question' | 'teacher_led',
  activeTeacherQuestionId?: string
}
```

**Questions**: Student submissions linked to sessions with like functionality
```typescript
{
  questionId: string,
  sessionId: string,
  text: string,
  studentId: string,  // Anonymous persistent ID
  isAnonymous: boolean,
  studentName?: string,
  likes?: {
    [studentId: string]: boolean  // Like tracking per student
  }
}
```

**Teacher Questions**: Teacher-prepared and real-time questions (NEW)
```typescript
{
  questionId: string,
  sessionId: string,
  text: string,
  teacherId: string,
  order: number,
  source: 'prepared' | 'realtime',
  status: 'waiting' | 'active' | 'completed',
  createdAt: number,
  activatedAt?: number,
  completedAt?: number
}
```

**Student Responses**: Responses to teacher questions (NEW)
```typescript
{
  responseId: string,
  questionId: string,  // Links to TeacherQuestion
  sessionId: string,
  studentId: string,
  text: string,
  createdAt: number,
  isAnonymous: boolean,
  studentName?: string
}
```

**AI Analysis Results**: Gemini API analysis of collected questions
```typescript
{
  clusteredQuestions: Array<{
    clusterId: number,
    clusterTitle: string,
    questions: string[],
    combinationGuide: string
  }>,
  recommendedActivities: Activity[],
  conceptDefinitions: Concept[]
}
```

### Critical Implementation Details

**Theme and Dark Mode System**
- Dual context system: `ThemeContext` for light/dark, `EducationLevelContext` for adaptive theming
- Theme system in `/src/styles/themes.ts` provides level-specific colors and styling
- Card components use Tailwind `dark:` classes for backgrounds
- **CRITICAL**: Always use `dark:text-white` for maximum contrast in dark mode - avoid `dark:text-gray-*`
- Use `useFullTheme()` hook to access complete theme object in components

**Firebase Security Rules**
- Teachers can only modify their own sessions
- Students can submit questions anonymously to any session
- Anonymous users can read sessions and submit questions/likes
- Questions have anonymous write access for student participation
- All advanced features (content sharing, analysis) require teacher authentication
- **NEW**: Teacher questions (`teacherQuestions/`) - teacher write-only, students read-only
- **NEW**: Student responses (`studentResponses/`) - students write, teachers read
- **NEW**: Question analyses (`questionAnalyses/`) - teacher write-only

**API Integration Pattern**
- `/src/app/api/ai/` contains Next.js API routes that proxy to Gemini
- Client sends encrypted API key in request headers
- Server-side API routes decrypt keys for Gemini API calls
- Never persist API keys on server side


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/reallygood83) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
