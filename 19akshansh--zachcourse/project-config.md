---
trigger: always_on
description: This document describes every AI agent in ZachCourse, how it works, what it uses, and where the code lives.
---

# ZachCourse — Agent Architecture

This document describes every AI agent in ZachCourse, how it works, what it uses, and where the code lives.

---

## Overview

ZachCourse is built around **seven specialized agents**, each responsible for a distinct part of the learning experience. They share a common infrastructure (Vercel AI SDK v7, Gemini via `@ai-sdk/google`, Zod schemas, NeonDB) but operate independently with no shared state at runtime.

```
User Request
     │
     ▼
Express Server (server.ts)
     │
     ├── POST /api/generate-roadmap   → Roadmap Agent
     ├── POST /api/generate-visual-roadmap → Visual Roadmap Agent
     ├── POST /api/generate-lesson    → Lesson Agent
     ├── POST /api/generate-quiz      → Quiz Agent
     ├── POST /api/mentor-chat        → Mentor Agent  ← agentic loop
     └── tRPC /api/trpc               → Progress Agent (DB reads/writes)
```

All routes protected by `requireAuth` middleware (Better Auth session check) and `aiRateLimit` (20 req/min per IP via `express-rate-limit`).

---

## Course Personalization Engine (Tone & Background Context)

ZachCourse implements a centralized personalization engine that dynamically alters the voice, formatting, and complexity of every single agent's response based on the learner's preferences:
1. **Personalization Fields:**
   - `experienceLevel`:🌱 Beginner, 🔥 Intermediate, 🚀 Advanced.
   - `backgroundContext`: A 500-character free-text field highlighting past programming languages, favorite learning styles, and goals.
   - `tone`: A 4-way style guideline configuration (Professional, Friendly, Gen Z, ELI5) stored in `src/lib/tone-options.ts`.
2. **Multi-Agent Threading:**
   - **Roadmap / Visual Roadmap Agents:** Adapts lesson topics, graph node counts, sequencing, and pacing parameters.
   - **Lesson Agent:** Adapts the explanation style, vocabulary complexity, and introductory pacing.
   - **Quiz Agent:** Shapes question scenarios, options, and explanations to mirror the chosen style.
   - **Mentor Agent:** Feeds system guidelines directly into conversational system prompts for adaptive tutee chats.
   - **Project Generation (generateProject):** Automatically extracts the course's `tone` and `backgroundContext` to customize the description, steps, and success criteria for module-end projects.

---

## Agent 1 — Roadmap Agent

**File:** `server.ts` → `POST /api/generate-roadmap`

**Purpose:** Takes a topic, URL, or raw syllabus text and generates a structured week-by-week learning roadmap.

**Technique:** `generateObject` with a strict Zod schema — the model is forced to return validated JSON matching the schema exactly. No free-text parsing required.

**Input:**
```typescript
{
  topic: string,          // max 500 chars
  sourceUrl?: string,     // optional reference URL
  textContent?: string,   // optional pasted syllabus, capped at 3000 chars in prompt
  experienceLevel: "beginner" | "intermediate" | "advanced",
  backgroundContext?: string, // optional learner background and preferences
  weeklyHours: number
}
```

**Output schema (Zod):**
```typescript
z.object({
  title: z.string(),
  description: z.string(),
  difficulty: z.string(),
  totalDuration: z.string().optional(),
  prerequisites: z.array(z.string()),
  modules: z.array(z.object({
    id: z.string(),
    title: z.string(),
    description: z.string(),
    lessons: z.array(z.object({
      id: z.string(),
      title: z.string(),
      duration: z.string(),
      concepts: z.array(z.string()),
      difficulty: z.string().optional(),
      type: z.string().optional(),
      description: z.string().optional(),
    }))
  }))
})
```

**Prompt rules:**
- 3–5 modules, each with 3–5 lessons
- Progressive difficulty
- Last lesson must be a hands-on project
- First lesson completable in under 30 minutes
- Depth adapts to `experienceLevel` ( beginner/intermediate/advanced tailoring)
- Course pacing and module density scale with `weeklyHours` commitment (ranging from casual to intensive)

**Fallback:** `getLocalFallbackRoadmap()` — returns a hardcoded template if all Gemini models fail.

**Model:** `callAI()` helper — tries `gemini-2.5-flash` → `gemini-2.0-flash` → `gemini-2.5-pro` in order.

---

## Agent 2 — Visual Roadmap Agent

**File:** `server.ts` → `POST /api/generate-visual-roadmap`

**Purpose:** Generates a full node-graph roadmap (20–35 nodes, 25–40 edges) rendered by `@xyflow/react` on the frontend.

**Technique:** `generateObject` with an extended Zod schema covering node types, edge types, resources, and difficulty levels.

**Output schema (key fields):**
```typescript
z.object({
  nodes: z.array(z.object({
    id: z.string(),
    type: z.enum(["start","module","lesson","milestone","project","end"]),
    label: z.string(),
    description: z.string(),
    duration: z.string().optional(),
    difficulty: z.enum(["Beginner","Intermediate","Advanced"]).optional(),
    concepts: z.array(z.string()).optional(),
    resources: z.array(z.object({
      title: z.string(),
      type: z.enum(["video","article","doc","practice"]),
      url: z.string().optional(),
    })).optional(),
  })),
  edges: z.array(z.object({
    id: z.string(),
    source: z.string(),
    target: z.string(),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [19akshansh/zachcourse](https://github.com/19akshansh/zachcourse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
