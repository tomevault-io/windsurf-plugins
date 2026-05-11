---
trigger: always_on
description: This file is for AI agents. Human contributors, see [CONTRIBUTING.md](CONTRIBUTING.md).
---

# Cmdr

This file is for AI agents. Human contributors, see [CONTRIBUTING.md](CONTRIBUTING.md).

Cmdr is an extremely fast AI-native file manager written in Rust, free forever for personal use on macOS (BSL license).
Downloadable at [the website](https://getcmdr.com).

- Dev server: `pnpm dev` at repo root
- Prod build: `pnpm build` at repo root

## Principles

These are general principles for the whole project. These are not just empty sentences on our wall, we live these:

### Top 5:

1. **Deliver delightful UX.** We always go the extra mile to make it absolutely delightful to use our software. Not just
   functional, but noticeably pleasant. Thoughtful phrasing. Accessible focus indicators. Putting real effort in
   dark/light modes. Nice images and anims. OS-custom everything. Respect the system font, sizing, theme,
   `prefers-reduced-motion`, etc.
2. **Elegance above all.** We have time to do outstanding work. We prefer a clean and elegant architecture over hacks,
   both internally for ourselves, and externally toward the user. We think about the long run.
3. **The app should feel rock solid.** The UI must _always_ be responsive. We never block the main thread. Every user
   action triggers immediate feedback, even if it's just a spinner. We communicate what's actually happening. Show
   progress. An ETA when possible. No progress bars stuck at 100% — show the real state. Long operations are always
   cancelable, stopping background work too, not just the UI. The user is always in control. Assume the hostile case
   (dead network mount, huge directory, crashed mid-operation) and handle it gracefully.
4. **Protect the user's data.** Use safe overwrite patterns like temp+rename. Offer rollback for destructive operations.
   Use atomic ops where possible. Design for the crash mid-operation. Test the shit out of the parts that write data.
5. **Be respectful to the user's resources.** Minimize CPU use, memory use, don't thrash the disks.

### Technicals:

1. **Think from first principles, capture intention.** Add logs. Run the code. Do benchmarks. Then document the "why"s
   and link the data where needed.
2. **Invest in finding the right tradeoff.** Elegance lives between duplication and overengineering. No premature
   abstractions, but no copy-paste either.
3. **Smart backend, thin frontend.** Complex logic lives in Rust. The frontend's job is to deliver a delightful UX:
   presenting the right states, errors, progress, and feedback. Display logic _can_ get complex, but the business logic
   lives in the backend.
4. **Organized by feature, not by layer.** Frontend components, backend modules, tests, and docs are colocated with the
   feature they belong to. Colocated `CLAUDE.md` files, colocated tests, feature-shaped directories. If we could
   technically merge a Svelte component with its Rust counterpart into one feature unit, we would.
5. **Thin IPC layer.** Tauri commands are pass-throughs. no branching, no transformation. Business logic lives in
   subsystem modules that can be tested independently.
6. **Subscribe, don't poll.** Whenever possible, we make it so that consumers can subscribe to events and receive
   updates. If not possible, we resort to polling. But we make an effort to avoid polling.
7. **Invest in testability.** We have virtual MTP devices, Docker-based SMB servers, feature flags for E2E. Tools to
   guarantee stability.
8. **Invest in tooling.** We have check runners, linters, coverage, CI. Tooling must be fast so we use it, and strict so
   it doesn't allow us to make mistakes.

## File structure

This is a monorepo containing these apps:

- Cmdr: Currently for macOS only. Rust, Tauri 2, Svelte 5, TypeScript, and custom CSS.
- Analytics dashboard: Private SvelteKit metrics dashboard. Deployed to Cloudflare Pages.
- getcmdr.com website: Astro + Tailwind v4. Deployed via Docker + Caddy.
- API server: Cloudflare Worker + Hono. Licensing, telemetry, crash reports, downloads, and admin endpoints.

Core structure:

- `/.github/workflows/` - GitHub Actions workflows
- `/apps/`
  - `analytics-dashboard/` - Private metrics dashboard (SvelteKit + CF Pages)
  - `desktop/` - The Tauri desktop app
    - `test/e2e-linux/` - WebDriverIO + tauri-driver tests (Docker, tests real Tauri app)
    - `src/` - Svelte frontend. Uses SvelteKit with static adapter. TypeScript strict mode. Custom CSS with design
      tokens.
      - `lib/` - Components
      - `routes/` - Routes
    - `src-tauri/` - Latest Rust, Tauri 2, serde, notify, tokio
    - `static/` - Static assets
    - `test/` - Vitest unit tests
  - `api-server/` - Cloudflare Worker (Hono). Licensing, telemetry, crash reports, downloads, and admin endpoints.
  - `website/` - Marketing website (getcmdr.com)
- `/scripts/check/` - Go-based unified check runner
- `/docs/` - Dev docs
  - `guides/` - How-to guides
  - `notes/` - Temporary reference notes (benchmarks, analysis) linked from CLAUDE.md files
  - `tooling/` - Internal tooling docs
  - `architecture.md` - Map of all subsystems with links to their `CLAUDE.md` files
  - `style-guide.md` - Writing, code, and design style rules
  - `security.md` - Security policies
- Feature-level docs live in **colocated `CLAUDE.md` files** next to the code (for example,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vdavid/cmdr](https://github.com/vdavid/cmdr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
