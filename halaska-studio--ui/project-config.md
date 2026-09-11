---
trigger: always_on
description: UX patterns and styled components for AI products — agents, copilots, and chat — by Halaska Studio.
---

# Halaska Kit

UX patterns and styled components for AI products — agents, copilots, and chat — by Halaska Studio.
Built on shadcn/ui foundations. Optimized for Claude artifact prototyping, portable to any React project.

## Project Structure

The kit is a **single self-contained file**: `halaska-kit-v1.0.jsx`. The `src/` folder is just a thin Vite entry that imports it.

```
halaska-kit/
├── CLAUDE.md              # This file — project context for Claude Code
├── package.json           # Vite + React (react + react-dom are the only deps)
├── src/
│   ├── main.jsx           # Vite entry — imports ../halaska-kit-v1.0.jsx
│   ├── tokens.js          # (reference copy of tokens)
│   ├── hooks.js           # (reference copy of hooks)
│   └── components/index.js # Component name index
└── halaska-kit-v1.0.jsx   # THE KIT — tokens, components, patterns, showcase page
```

## Showcase Page Order & Navigation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Halaska-Studio/ui](https://github.com/Halaska-Studio/ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
