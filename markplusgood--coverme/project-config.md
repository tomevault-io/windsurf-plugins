---
trigger: always_on
description: This playbook guides AI coding agents working on **cover.me**, an AI tool that generates tailored cover letters from a user’s background, resume, and job description. It keeps patterns consistent, enforces security and accessibility, and aligns the product toward high conversion and quality.
---

## Introduction

This playbook guides AI coding agents working on **cover.me**, an AI tool that generates tailored cover letters from a user’s background, resume, and job description. It keeps patterns consistent, enforces security and accessibility, and aligns the product toward high conversion and quality.

---

## Quick Reference Card

```
PROJECT: cover.me — AI cover letter writer
STACK: SvelteKit + TypeScript + Tailwind + shadcn-svelte
DEPLOYMENT: Cloudflare Pages
AUTH: Supabase Auth
DATABASE: Supabase (user data) + Cloudflare D1 (cache of model outputs)
STORAGE: Cloudflare R2 (resume uploads, exports)
PACKAGE MANAGER: Bun
```

---

## 📋 CRITICAL: Development Process Requirements

**BEFORE ANY IMPLEMENTATION, YOU MUST:**

1. **Read the canonical documentation in this exact order:**
  
  1. devdocs/prd.md
  2. devdocs/planning/development-roadmap.md
  3. AGENTS.md

2. **Follow the Stage Implementation Protocol:**
   - Review current stage requirements from `devdocs/planning/development-roadmap.md`
   - Create implementation plan with testable success criteria
   - Implement ONLY current stage features (no premature implementation)
   - Validate against documented success criteria before claiming completion

3. **Stage Completion Requirements:**
   - All stage requirements from roadmap must be implemented
   - Success criteria must be met and testable
   - Code must follow AGENTS.md principles
   - Documentation must be updated

---

## Core Principles

1) **Mobile-first**: design for small screens first, then add breakpoints.
2) **Accessibility**: every interactive element must be keyboard and screen-reader friendly (ARIA labels, focus states).
3) **Type safety**: strict TypeScript; no `any`. Model all payloads (job, resume, letter).
4) **Server-side validation**: validate/sanitize job text, resumes, and preferences on the server.
5) **Privacy & PII**: never log raw resumes or job descriptions; strip PII from telemetry.
6) **ATS-friendly output**: clean text structure, single column, no decorative images in exports.
7) **Conversion mindset**: propose A/B tests when UX changes could affect signup, generation, or upgrade funnels.
8) **Documentation First**: Review product requirements in `devdocs/prd.md` and reference feature-specific documents in `devdocs` before planning and implementing any new feature.

### Mobile-First Example
```svelte
<!-- ✅ Correct -->
<div class="flex flex-col gap-4 md:flex-row md:gap-6 lg:gap-8">
```

### Accessible Button Example
```svelte
<Button on:click={generate} aria-label="Generate cover letter" class="focus:ring-2 focus:ring-primary-500">
  Generate
</Button>
```

### Typed Payload Example
```typescript
interface CoverLetterPayload {
  userId: string;
  jobTitle: string;
  company: string;
  jobDescription: string;
  resumeUrl?: string;
  tone: 'concise' | 'enthusiastic' | 'formal';
  language?: string;
}
```

---

## Project Structure Guide

- Components: `PascalCase.svelte` (e.g., `LetterPreview.svelte`, `ResumeUpload.svelte`)
- Routes: lowercase `+page.svelte`, `+page.server.ts`, `+layout.svelte`
- Utils: `kebab-case.ts` (e.g., `job-parser.ts`, `resume-reader.ts`)
- Types: `PascalCase.ts` (e.g., `CoverLetter.ts`, `UserProfile.ts`)
- API routes: `+server.ts`

### Import Order
```typescript
// 1. External deps
import { z } from 'zod';
// 2. SvelteKit
import { error, redirect } from '@sveltejs/kit';
// 3. Internal absolute
import { createCoverLetter } from '$lib/server/ai/letters';
// 4. Relative
import Form from './Form.svelte';
// 5. Types-only
import type { PageServerLoad } from './$types';
```

---

## Component Patterns

### Standard Svelte Component
```svelte
<script lang="ts">
  import { Button } from '$lib/components/ui/button';
  import type { CoverLetterDraft } from '$lib/types/CoverLetter';

  export let draft: CoverLetterDraft | null = null;
  let loading = false;

  async function regenerate() {
    loading = true;
    await triggerRegeneration();
    loading = false;
  }
</script>

<section class="space-y-4">
  <header class="flex items-center justify-between gap-3">
    <h2 class="text-xl font-semibold text-neutral-900">Preview</h2>
    <Button size="sm" on:click={regenerate} disabled={loading} aria-label="Regenerate cover letter">
      {loading ? 'Working…' : 'Regenerate'}
    </Button>
  </header>

  {#if draft}
    <article class="prose max-w-none bg-white p-4 rounded-lg border border-neutral-200">
      {@html draft.html}
    </article>
  {:else}
    <p class="text-sm text-neutral-600">No draft yet. Add a job description to start.</p>
  {/if}
</section>
```

### Protected Layout Example
```typescript
// src/routes/app/+layout.server.ts
import { redirect } from '@sveltejs/kit';
import type { LayoutServerLoad } from './$types';

export const load: LayoutServerLoad = async ({ locals }) => {
  const { user } = await locals.safeGetSession();
  if (!user) throw redirect(302, '/auth/login');
  return { user };
};
```

### Dashboard Loader Example
```typescript
// src/routes/app/+page.server.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/markplusgood) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
