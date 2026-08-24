---
trigger: always_on
description: !IMPORTANT: Please update @AGENTS.md and @ARCHITECTURE.md after each big change to reflect changes
---

# BetterLectio

!IMPORTANT: Please update @AGENTS.md and @ARCHITECTURE.md after each big change to reflect changes

**Design skill:** When building big new features that require design, or doing significant UI changes/refactors, use the `frontend-design` skill to generate high-quality, polished interfaces. Always invoke it for new page redesigns, component overhauls, or visual reworks.

@ARHITECTURE.md

Browser extension that modernizes [Lectio](https://www.lectio.dk/), a Danish school management system.

## Tech Stack
- **WXT** - Browser extension framework
- **Preact** - Lightweight React alternative (aliased from React)
- **TypeScript** + **Tailwind CSS**
- **shadcn/ui** + **Radix UI** - UI components

## Key Files

### Entry Points
- `entrypoints/content.tsx` - Main content script, renders custom UI wrapper, injects page-specific components
- `entrypoints/login.content.tsx` - Login page redesign with school selector
- `entrypoints/hide-flash.content.ts` - FOUC prevention + intercepts Lectio CSS into @layer lectio
- `entrypoints/elevfeedback-frame.content.ts` - `document_start` + `all_frames` chrome strip for the Elevfeedback editor iframe (`window.name === bl-elevfeedback-editor`). Do not use srcdoc — CKEditor and ASP.NET save need the live Lectio page.
- `entrypoints/session-block.content.ts` - Blocks session timeout popup

### Components
- `components/AppSidebar.tsx` - Default sidebar navigation with collapsible sections; student name/avatar prefer Supabase `name` and `custom_pfp_url`/`lectio_pfp_url` before Lectio DOM data
- `components/HorizontalNavbar.tsx` - Opt-in desktop top navigation (`interface.navigationLayout = "horizontal"`). Primary Forside/Skema/Elever/Beskeder links respect navigation settings; More reflects active secondary pages. Quick actions use tooltips and collapse into overflow before narrow-desktop collisions. Live Lectio context rows retain page-specific navigation, strip redundant active-section suffixes, and add history-aware back links for viewed entities. Its rails share the 110rem wide-screen cap used by Forside. The sidebar remains the default.
- `components/AppOverlays.tsx` - Layout-independent owner for Settings, onboarding, activity/private-appointment dialogs, the assignment detail sheet, and the Elevfeedback editor overlay. Keep these outside either navigation surface so global events work in both layouts.
- `components/OnboardingWizard.tsx` - First-run setup for theme, navigation layout, subject colors, optional profile details, and a mobile-app QR pitch (`renderMobileAppQrSvg`). Navigation asks users to choose between the default/recommended sidebar and the Lectio-like top menu; finishing reloads only when the mounted navigation shell must change. The app step is skipped when `app_installed_at` is already set.
- `components/FindSkemaPage.tsx` - FindSkema redesign with fuzzy search, starred/recents, person cards, Supabase-backed student avatars, and student search matching both Lectio names and Supabase preferred names
- `components/ProfilePage.tsx` - Student profile with tabbed skema/classmates/teachers/hold/dokumenter views. Supabase-backed: description, instagram, birthday (if `show_birthday`), BL badge. Own-profile inline edit form.
- `components/ProfilPage.tsx` - Logged-in student's profile editor. Includes the referral-unlocked custom profile-picture card: locked progress, private upload preview, moderation status/rejection feedback, and approval cooldown.
- `lib/instagram.ts` - Shared Instagram helpers that accept `handle`, `@handle`, or pasted URLs and normalize to `@handle` display/link
- `components/PersonCard.tsx` - Reusable person/entity card with lazy-loaded pictures, navigation context (`from`, `q`, `name`), optional BL badge, Supabase-first name/avatar resolution
- `components/DokumenterPage.tsx` - Documents redesign: collapsible folder tree (hold colors), file list with extension-based icons/badges, breadcrumbs, search, in-app image/PDF preview, drag-and-drop upload, create folder, sort. Parses native DOM via `lib/dokumenter-parser.ts`
- `components/ViewingScheduleHeader.tsx` - Header when viewing another schedule (star/back + expandable "Medlemmer" panel)
- `components/LektierPage.tsx` - Day-grouped homework cards with Supabase-backed done-state sync (optimistic local toggle, cross-device persistence)
- `components/OpgaverPage.tsx` - Single chronological timeline of all assignments grouped by week, auto-scrolls to current week, compact rows with status indicators, fravær badges, hold pills, grade badges, ignore-missing toggle, combined elevtimer per week
- `components/OpgaveDetailSheet.tsx` - Assignment detail side sheet with submission history, comment/file upload, and Supabase-first group-member names/avatars. Always mounted by `AppOverlays`; listens for `betterlectio:openOpgaveDetail` event so any surface (incl. schedule deadline bricks) can open it without navigation.
- `components/BeskederThreadView.tsx` - Thread view with Supabase-first sender names/avatars, WYSIWYG reply, no-reload reply/attach, and Lectio-native message reactions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonbng/betterlectio](https://github.com/jonbng/betterlectio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
