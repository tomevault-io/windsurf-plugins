---
trigger: always_on
description: A React SPA that fetches and displays Nogizaka46 member blogs from the official website, with AI-powered Japanese→English/Vietnamese translation using Gemini API. Built with Vite + React 19 + Ant Design Pro, deployed on Vercel.
---

# Nogizaka46 Blog Archive - AI Coding Instructions

## Project Overview
A React SPA that fetches and displays Nogizaka46 member blogs from the official website, with AI-powered Japanese→English/Vietnamese translation using Gemini API. Built with Vite + React 19 + Ant Design Pro, deployed on Vercel.

## Architecture & Critical Patterns

### Dual Proxy Architecture (Critical!)
The app uses **two different proxy systems** depending on environment:
- **Development**: Local Express proxy (`local-proxy-server.js` on port 3001) + Vite dev server proxy
- **Production**: Vercel serverless function (`api/proxy.js`) 
- **Why**: iOS Safari CORS blocks direct requests to `nogizaka46.com`. Proxy adds proper User-Agent and CORS headers.
- **Always use proxy**: `shouldUseProxy()` in `src/utils/deviceDetection.js` returns `true` for all environments

### Data Flow Pattern
1. **API calls flow**: Component → `blogService.js` → Device detection → Either `fetchWithProxy()` (iOS/production) OR direct `axios` (fallback)
2. **JSONP parsing**: Official site returns JSONP (`res({...})`). Strip wrapper: `.replace(/^res\(/, "").replace(/\);?$/, "")` before `JSON.parse()`
3. **Caching**: In-memory `Map` caches in `blogService.js` (10min TTL for members, permanent for blog details)

### API Structure (Official Nogizaka46)
- **Member List API**: `https://www.nogizaka46.com/s/n46/api/list/member?callback=res`
  - Returns array of members with `code`, `name`, `cate` (generation), `img`, `graduation`, etc.
  - Key field: `code` (member code, e.g., "36758")
  
- **Blog List API**: `https://www.nogizaka46.com/s/n46/api/list/blog?ima={timestamp}&rw={limit}&st={offset}&arti_code={memberCode}&callback=res`
  - `ima`: Unix timestamp (use `Math.floor(Date.now() / 1000)`)
  - `rw`: Results per page (default 32)
  - `st`: Offset for pagination (0, 32, 64...)
  - `arti_code`: Member code from member API (filters by member)
  - Returns array with `code` (blog ID), `title`, `text` (HTML), `img`, `date`, `arti_code`, `name` (author)
  
- **Blog Detail API**: `https://www.nogizaka46.com/s/n46/api/list/blog?ima={timestamp}&cd={blogId}&callback=res`
  - `cd`: Blog code/ID
  - Returns single-item array with full blog details

**Optimization principle**: Keep code simple, use direct API calls with proper params, minimize redundant logic.

### iOS 18+ Specific Workarounds
- **Extended timeouts**: 25s for iOS 18+, 20s for others (see `fetchWithProxy` in `src/api/proxy.js`)
- **Exponential backoff**: Retry with `Math.pow(2, attempt) * 1000 + attempt * 500` delay
- **Special headers**: `Sec-Fetch-*` headers required for iOS Safari (in `proxy.js`)
- **Known issue**: iOS detail page loading is buggy (see README roadmap)

### Translation System
- **Streaming translations**: Use `onProgress` callback in `GeminiTranslate.js` to update UI incrementally
- **Chunking**: Text split into 4000-char chunks with HTML tag preservation (`splitTextIntoChunks`)
- **Vietnamese style guide**: Use "mình" (I/me), "mọi người" (fans), never "ạ/nhé". Keep idol diary tone.
- **Title cleaning**: `cleanTitleTranslation()` removes instruction artifacts from Gemini responses

## Development Workflows

### Starting Development
```bash
npm run dev:full   # Starts BOTH proxy server AND Vite (required for iOS testing)
npm run dev        # Vite only (won't work on iOS/Safari without proxy)
npm run proxy      # Proxy only (for debugging proxy issues)
```

**Never use `npm run dev` alone** - the proxy is required for the app to work correctly.

### Build & Deploy
```bash
npm run build      # Vite production build with Terser minification
npm run preview    # Test production build locally
```

**Vercel config**: See `vercel.json` for:
- API routes timeout (30s)
- SPA fallback rewrite rules (important for React Router)
- CORS headers for `/api/*` routes

### Environment Variables
Required in `.env`:
```env
VITE_GEMINI_API_KEY=your_key_here
```

Accessed via `import.meta.env.VITE_*` (Vite convention). See `src/config/env.js` for validation.

## Component Patterns

### Mobile/Desktop Duality
- Desktop: `BlogList.jsx`, `BlogDetail.jsx`  
- Mobile: `BlogListMobile.jsx`, `BlogDetailMobile.jsx`
- Routing: Conditional render based on `window.innerWidth < 768` (see `App.jsx` routes)
- Layout: Desktop uses `ProLayout` sidebar with `MemberProfile`, mobile uses `PageContainer`

### Styling Approach
- **Primary**: Ant Design Pro components + ConfigProvider theming (see `App.jsx` `tokens` and `componentTokens`)
- **Secondary**: Tailwind utility classes (configured to not conflict: `preflight: false` in `tailwind.config.js`)
- **Theme**: "book-background" aesthetic with light/dark modes. Colors: `#8B4513` (brown) light, `#9c6b3f` dark
- **Japanese fonts**: Auto-detected in `optimizeHtmlContent()` - applies `'Noto Sans JP'` stack to Japanese text

### i18n Pattern
- **No library**: Manual translation objects like `const t = { searchPlaceholder: { ja: "...", en: "...", vi: "..." } }`
- **Access**: `t.searchPlaceholder[language]`
- **State**: Language stored in component state, passed to children as props

## File Locations & Responsibilities


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NgocThachTN/Nogizaka46BlogArchived](https://github.com/NgocThachTN/Nogizaka46BlogArchived) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
