---
trigger: always_on
description: LOL URL is a modern URL shortener built with **React (Vite) + Supabase + Tailwind CSS**. Users can shorten URLs, generate QR codes, and track detailed analytics (clicks, location, device stats). Authentication is handled via Supabase Auth.
---

# LOL URL - AI Coding Agent Instructions

## Project Overview
LOL URL is a modern URL shortener built with **React (Vite) + Supabase + Tailwind CSS**. Users can shorten URLs, generate QR codes, and track detailed analytics (clicks, location, device stats). Authentication is handled via Supabase Auth.

## Architecture

### Tech Stack
- **Frontend**: React 18, Vite, React Router v7, Tailwind CSS + shadcn/ui components
- **Backend**: Supabase (PostgreSQL database, Auth, Storage for QR codes/profile pics)
- **Deployment**: Vercel
- **Styling**: Tailwind with custom design system (see [tailwind.config.js](../tailwind.config.js))

### Key Patterns

#### 1. Database Access Layer (`src/db/`)
All Supabase operations are abstracted into dedicated API files:
- `apiAuth.js` - Authentication (login, signup, getCurrentUser, logout)
- `apiUrls.js` - URL CRUD operations (getUrls, createUrl, deleteUrl, getLongUrl)
- `apiClicks.js` - Analytics tracking (getClicksForUrls, storeClicks)
- `supabase.js` - Supabase client initialization with env vars

**Pattern**: Never call `supabase` directly in components. Always use these API functions.

#### 2. Custom `useFetch` Hook (`src/hooks/use-fetch.jsx`)
Wrapper for async operations with loading/error states:
```jsx
const {data, loading, error, fn} = useFetch(apiFunction);
// Call fn(...args) to execute, data/loading/error update automatically
```
**Usage**: All async operations (DB calls, auth) should use this hook for consistent state management.

#### 3. Global Auth Context (`src/context.jsx`)
- Provides `user`, `fetchUser`, `loading`, `isAuthenticated` via `UrlState()` hook
- Authentication check: `user?.role === "authenticated"`
- Always use `UrlState()` instead of direct Supabase auth calls

#### 4. Component Structure
- **Pages**: [src/pages/](../src/pages/) (landing, dashboard, auth, link, redirect-link)
- **Shared Components**: [src/components/](../src/components/) (create-link, link-card, location-stats, device-stats)
- **UI Components**: [src/components/ui/](../src/components/ui/) (shadcn/ui - button, dialog, input, etc.)
- **Layouts**: [src/layouts/app-layout.jsx](../src/layouts/app-layout.jsx) (contains Header, handles navigation)

**Import Alias**: Use `@/` for src imports (configured in [vite.config.js](../vite.config.js))

#### 5. Routing (React Router v7)
- Defined in [src/App.jsx](../src/App.jsx) with `createBrowserRouter`
- Protected routes use `<RequireAuth>` wrapper component
- Custom URL redirects handled by [redirect-link.jsx](../src/pages/redirect-link.jsx) and [redirect-handler.jsx](../src/components/redirect-handler.jsx)

## Development Workflow

### Running Locally
```bash
npm run dev        # Standard dev server
npm run chala      # Alias for dev (cultural preference, don't remove)
npm run build      # Production build
npm run preview    # Preview production build
```
Dev server runs on `http://localhost:5173` (Vite default, NOT 3000)

### Environment Variables
Required in `.env`:
```
VITE_SUPABASE_URL=your_supabase_url
VITE_SUPABASE_KEY=your_supabase_anon_key
```
**Critical**: All env vars must be prefixed with `VITE_` to be accessible in client code via `import.meta.env.VITE_*`

### Database Schema
See [docs/database.md](../docs/database.md) for complete schema. Key tables:
- `urls` - Stores original_url, short_url, custom_url, qr (QR code image URL), user_id
- `clicks` - Analytics data (country, city, device info)
- Supabase Storage buckets: `qrs` (QR codes), `profile-pic` (user avatars)

## Code Conventions

### Styling
- Use Tailwind utility classes (no inline styles or CSS modules)
- UI components follow shadcn/ui patterns with `cn()` utility ([src/lib/utils.js](../src/lib/utils.js))
- Toast notifications: Use `react-toastify` with config from [src/utils/toastConfig.js](../src/utils/toastConfig.js)
- Theme: Dark mode preferred, uses CSS variables defined in [src/index.css](../src/index.css)

### Error Handling
```jsx
// Standard pattern in components:
if (error) return <Error message={error.message} />;
if (loading) return <BeatLoader />; // from react-spinners
```

### Form Validation
- Use `yup` for schema validation (see [create-link.jsx](../src/components/create-link.jsx) L15 for example)
- Display errors via `<Error>` component with field-specific messages

### QR Code Generation
- Use `react-qrcode-logo` library (see [create-link.jsx](../src/components/create-link.jsx))
- QR codes stored in Supabase Storage, URL saved to `urls.qr` field
- QR customization: Colors, logos, sizes all configurable (extensive UI in CreateLink component)

## Important Implementation Details

### URL Shortening Logic
1. Generate random 4-char short_url: `Math.random().toString(36).substring(2, 6)`
2. Allow optional custom_url (check uniqueness via `checkCustomUrlExists`)
3. Store QR code to Supabase Storage first, then insert URL record
4. Short URL format: `{domain}/{short_url or custom_url}`

### Redirect Handling
- [redirect-link.jsx](../src/pages/redirect-link.jsx) fetches long URL by short code
- [redirect-handler.jsx](../src/components/redirect-handler.jsx) logs click analytics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rohit-Dnath/LOL-URL](https://github.com/Rohit-Dnath/LOL-URL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
