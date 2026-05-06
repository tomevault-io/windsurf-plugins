---
trigger: always_on
description: This project uses **br** (beads) for issue tracking. Run `br onboard` to get started.
---

# Agent Instructions

# AGENTS.md — Platform-Optimized SvelteKit PWA (Mobile + Tablet + Desktop)

This project uses **br** (beads) for issue tracking. Run `br onboard` to get started.

## Project Structure

```
gastown-ui/                          # Town root
├── .beads/                          # Town-level beads (mayor mail, HQ coordination)
├── AGENTS.md                        # This file
├── CLAUDE.md                        # Mayor context
│
└── gastown-ui/                      # Rig: SvelteKit UI project
    ├── .repo.git/                   # Bare git repository (shared by all worktrees)
    │
    ├── mayor/
    │   ├── rig/                     # Mayor's reference clone (READ-ONLY)
    │   │   └── .beads/              # Rig beads database (source of truth)
    │   └── state.json
    │
    ├── polecats/                    # Worker worktrees
    │   ├── furiosa/                 # Polecat worktree
    │   │   └── .beads/redirect      # → ../../mayor/rig/.beads
    │   └── nux/                     # Polecat worktree
    │       └── .beads/redirect      # → ../../mayor/rig/.beads
    │
    ├── refinery/
    │   └── rig/                     # Merge queue processor worktree (on main)
    │       └── .beads/redirect      # → ../../mayor/rig/.beads
    │
    ├── crew/                        # Human-managed worktrees
    │   └── amrit/
    │
    └── witness/                     # Polecat lifecycle monitor
        └── state.json
```

### Key Concepts

| Component | Purpose |
|-----------|---------|
| **Town** | Workspace root containing all rigs |
| **Rig** | Project container (one per repo) |
| **Polecat** | Worker agent with isolated git worktree |
| **Refinery** | Merge queue processor (rebases, tests, merges) |
| **Witness** | Monitors polecat lifecycle |
| **Beads** | Issue tracking, shared via redirect to mayor/rig/.beads |

## Quick Reference

```bash
br ready              # Find available work
br show <id>          # View issue details
br claim <id>         # Claim work (sets assignee, derives in_progress display status)
br close <id>         # Complete work
br sync               # Sync with git
```

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   br sync
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

---

## Critical Fixes Phase - Patterns & Discoveries

### Form Validation Pattern

**When**: All user forms need to validate input before submission

**Pattern**:
```typescript
import { z } from 'zod';

// 1. Define validation schema
const formSchema = z.object({
  title: z.string().min(3, 'Title must be at least 3 characters'),
  type: z.enum(['task', 'bug', 'feature', 'epic']),
  priority: z.number().min(0).max(4)
});

// 2. In form handler, validate BEFORE submission
async function handleSubmit(e: Event) {
  e.preventDefault();
  errors = {};
  
  const result = formSchema.safeParse({
    title: formTitle,
    type: selectedType,
    priority: selectedPriority
  });
  
  if (!result.success) {
    const fieldErrors = result.error.flatten().fieldErrors;
    errors = Object.fromEntries(
      Object.entries(fieldErrors).map(([key, msgs]) => [key, msgs?.[0] || ''])
    );
    hapticError();
    return; // Don't submit
  }
  
  // Continue with submission
}

// 3. Display errors inline
{#if errors.title}
  <p class="text-sm text-destructive">{errors.title}</p>
{/if}

// 4. Disable submit until valid
<button disabled={formTitle.length < 3}>Submit</button>
```

**Applied To**:
- Work page: Issue creation, Convoy creation, Sling work forms

**Key Points**:
- Use Zod for client-side validation (npm install zod)
- Validate on submit, not on blur
- Show specific error messages per field
- Disable submit button while invalid
- Clear errors on successful submission

---

### Error State Integration Pattern

**When**: Any page that fetches data asynchronously

**Pattern**:
```svelte
<script>
  let error: Error | null = null;
  let loading = true;
  
  async function fetchData() {
    try {
      const res = await fetch('/api/data');
      if (!res.ok) throw new Error('Failed to fetch');
      data = await res.json();
    } catch (e) {
      error = e instanceof Error ? e.message : 'Unknown error';
    } finally {
      loading = false;
    }
  }
  
  function handleRetry() {
    error = null;
    fetchData();
  }
  
  onMount(() => {
    fetchData();
  });
</script>

{#if loading}
  <SkeletonCard type="mail" count={5} />

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Avyukth/gastown_ui](https://github.com/Avyukth/gastown_ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
