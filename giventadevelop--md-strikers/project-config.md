---
trigger: always_on
description: This rule defines the **mandatory** pattern for data fetching on the home page (and any public-facing landing page). All API / database calls MUST be deferred until **after** the browser has completed the initial paint of static content. This eliminates network request bursts that compete with rendering during hydration, producing a significantly faster perceived load — especially critical for AWS Amplify Lambda deployments where cold starts already add latency.
---

# Homepage Deferred Loading Pattern — Faster Initial Page Load on AWS Amplify

## Overview

This rule defines the **mandatory** pattern for data fetching on the home page (and any public-facing landing page). All API / database calls MUST be deferred until **after** the browser has completed the initial paint of static content. This eliminates network request bursts that compete with rendering during hydration, producing a significantly faster perceived load — especially critical for AWS Amplify Lambda deployments where cold starts already add latency.

## Problem Solved

- **Slow initial page load**: On first visit (cold start), multiple API calls fire simultaneously during React hydration, competing with the browser's paint and blocking the main thread with JSON parsing and state updates.
- **Network request burst**: 7+ concurrent API calls overwhelm the backend and the browser's connection limit (6 per origin in HTTP/1.1), queueing requests and delaying everything.
- **Static content blocked**: Users see a blank/loading page instead of immediately visible static sections (Services, About, Causes, etc.) because the main thread is busy processing API responses.

## Core Architecture

### Hook: `usePageReady` (`src/hooks/usePageReady.ts`)

Returns `true` after the browser has completed the initial paint cycle using nested `requestAnimationFrame`. This is the foundation for all deferred loading.

### Hook: `useDeferredFetch(delayMs)` (`src/hooks/usePageReady.ts`)

Returns `true` after `usePageReady()` + an additional configurable delay. Different delays stagger API calls so they don't fire simultaneously.

### Staggered Delay Tiers

| Priority | Delay | Component | Reason |
|----------|-------|-----------|--------|
| 0 (gate) | Page ready (~30ms) | `TenantSettingsProvider` | Determines section visibility; other sections depend on it |
| 1 (above fold) | 500ms | `HeroSection`, `LiveEventsSection`, `FeaturedEventsSection` | Above the fold, visible early but static fallback works |
| 2 (mid page) | 300ms after mount | `UpcomingEventsSection` | Mounts after TenantSettings loads (natural extra delay) |
| 3 (lower) | 800ms after mount | `TeamSection` | Further down page, mounts after TenantSettings |
| 4 (bottom) | 1500ms | `OurSponsorsSection` | Bottom of page, lowest priority |

### Timeline (First Visit, Cold Start)

```
0ms     — Server-side layout runs (auth checks on Amplify Lambda)
~500ms  — HTML arrives at browser, JS bundles start loading
~800ms  — React hydrates, static sections paint immediately
~830ms  — usePageReady fires → TenantSettingsProvider starts fetch
~1000ms — TenantSettings response → UpcomingEvents + Team mount
~1300ms — Hero/Live/Featured events data starts fetching
~1300ms — UpcomingEventsSection starts fetching (300ms after mount)
~1800ms — TeamSection starts fetching (800ms after mount)
~2330ms — OurSponsorsSection starts fetching
```

### Timeline (Repeat Visit, Cached)

```
0ms     — Page hydrates, static sections paint
~30ms   — Cache checks fire instantly → all cached data loads
          No network requests needed!
```

## Mandatory Pattern

### For Components That Fetch Data on the Home Page

```tsx
// ✅ DO: Always use deferred fetch pattern
import { useDeferredFetch } from '@/hooks/usePageReady';

const MySection: React.FC = () => {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(true);

  // Choose delay based on section position (see tier table above)
  const shouldFetch = useDeferredFetch(800);

  useEffect(() => {
    async function loadData() {
      // 1. ALWAYS check sessionStorage cache first (instant, no delay)
      try {
        const cached = sessionStorage.getItem(CACHE_KEY);
        if (cached) {
          const { data, timestamp } = JSON.parse(cached);
          if (Date.now() - timestamp < CACHE_DURATION) {
            setData(data);
            setLoading(false);
            return; // Cache hit — no network request needed
          }
        }
      } catch { /* ignore */ }

      // 2. Gate network request behind deferred flag
      if (!shouldFetch) return;

      // 3. Now safe to make API call
      const response = await fetch('/api/proxy/...');
      // ...
    }

    loadData();
  }, [shouldFetch]); // Re-run when shouldFetch becomes true
};
```

```tsx
// ❌ DON'T: Fetch immediately on mount
useEffect(() => {
  fetch('/api/proxy/...');
}, []); // Fires during hydration — blocks static content!
```

### For Shared Hooks (useEventsData)

```tsx
// ✅ DO: Accept `enabled` parameter to defer
export const useEventsData = (enabled: boolean = true) => {
  useEffect(() => {
    if (!enabled) return; // Don't fetch until enabled
    fetchEventsData();
  }, [enabled]);
};

// In the component:
const shouldFetch = useDeferredFetch(500);
const { filteredEvents } = useFilteredEvents('hero', shouldFetch);
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
