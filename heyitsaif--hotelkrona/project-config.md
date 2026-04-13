---
trigger: always_on
description: > This file provides comprehensive guidelines for Cursor IDE to understand our project structure, coding standards, and development workflow. It helps Cursor provide better suggestions and maintain consistency.
---

# Hotel Corona - Cursor IDE Rules & Guidelines

> This file provides comprehensive guidelines for Cursor IDE to understand our project structure, coding standards, and development workflow. It helps Cursor provide better suggestions and maintain consistency.

## 📋 Project Context

### Project Type
**Type**: Next.js 14 Hotel Website  
**Purpose**: Luxury coastal hotel website with booking functionality  
**Status**: Phase 1 Complete - Static site with UI ready for backend integration  
**Design Theme**: Coastal Castle Elegance

### Tech Stack Overview
```
Frontend: Next.js 14 (App Router) + TypeScript + Tailwind CSS
Animation: Framer Motion
Deployment: Vercel
Current Phase: Static website (no backend yet)
Next Phase: CMS integration + Booking API + Email service
```

## 🎨 Design System

### Color Palette
Our entire design is based on this coastal castle color scheme:

```typescript
// Coastal Colors (Primary Palette)
{
  'coastal-cream': '#e9d9c3',      // Light backgrounds, hero sections
  'coastal-sand': '#d1c4a8',        // Alternating backgrounds
  'coastal-gold-light': '#b39a6b',  // Borders, subtle accents
  'coastal-gold': '#c4a65a',        // Primary accent, hover states
  'coastal-bronze': '#8a6e3d',      // Body text, dark elements
}

// Accent Colors (Secondary Palette)
{
  'accent-burgundy': '#7d2e2e',     // CTA buttons, urgent actions
  'accent-dark-brown': '#3d2817',   // Headers, strong text
}
```

**Usage Rules:**
- Use `coastal-cream` for primary backgrounds
- Use `coastal-sand` for alternating sections
- Use `accent-burgundy` for all CTA buttons
- Use `coastal-bronze` for body text
- Use `coastal-gold` for hover effects and highlights

### Typography System

```typescript
// Font Families
Headings: 'Playfair Display', Georgia, serif
Body: 'Montserrat', system-ui, sans-serif

// Size Scale
h1: 3rem (48px)    - Hero headings
h2: 2.25rem (36px) - Section headings
h3: 1.875rem (30px) - Subsection headings
h4: 1.5rem (24px)  - Card headings
body: 1rem (16px)  - Regular text
small: 0.875rem (14px) - Captions, labels
```

### Spacing Scale
Follow Tailwind's scale consistently:
```
xs: 4px   sm: 8px   md: 12px   lg: 16px
xl: 24px  2xl: 32px  3xl: 48px  4xl: 64px
```

## 📁 Project Structure & File Organization

```
hotel-corona/
├── src/
│   ├── app/                      # Next.js App Router pages
│   │   ├── page.tsx             # Homepage (/)
│   │   ├── layout.tsx           # Root layout (wraps all pages)
│   │   ├── globals.css          # Global styles
│   │   ├── rooms/               # Rooms pages
│   │   │   ├── page.tsx        # /rooms (listing)
│   │   │   └── [slug]/page.tsx # /rooms/[slug] (details)
│   │   ├── pricing/page.tsx     # /pricing
│   │   ├── amenities/page.tsx   # /amenities
│   │   ├── about/page.tsx       # /about
│   │   ├── contact/page.tsx     # /contact
│   │   ├── gallery/page.tsx     # /gallery
│   │   └── booking/page.tsx     # /booking
│   │
│   ├── components/
│   │   ├── layout/              # Global layout components
│   │   │   ├── Header.tsx       # Navigation header
│   │   │   ├── Footer.tsx       # Site footer
│   │   │   ├── BookingBar.tsx   # Sticky booking bar
│   │   │   └── LoadingOverlay.tsx # Initial loader
│   │   │
│   │   ├── sections/            # Page-specific sections
│   │   │   ├── HeroSection.tsx
│   │   │   ├── WelcomeSection.tsx
│   │   │   ├── FeaturedRooms.tsx
│   │   │   ├── AmenitiesOverview.tsx
│   │   │   ├── TestimonialsSection.tsx
│   │   │   ├── RestaurantsSection.tsx
│   │   │   └── CTASection.tsx
│   │   │
│   │   └── ui/                  # Reusable UI components
│   │       ├── Button.tsx
│   │       ├── Card.tsx
│   │       ├── Input.tsx
│   │       └── RoomCard.tsx
│   │
│   └── lib/                     # Utilities and constants
│       ├── constants.ts         # Static data (rooms, amenities, etc.)
│       └── utils.ts            # Helper functions
│
└── public/                      # Static assets
    └── images/                  # Images organized by section
```

### File Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| **Components** | PascalCase | `HeroSection.tsx`, `BookingBar.tsx` |
| **Pages** | kebab-case | `[slug]/page.tsx`, `pricing/page.tsx` |
| **Utilities** | camelCase | `utils.ts`, `constants.ts` |
| **Constants** | UPPER_SNAKE_CASE | `MAX_GUESTS`, `ROOM_TYPES` |

## 💻 Code Standards & Patterns

### TypeScript Guidelines

#### 1. Always Define Types
```typescript
// ❌ Bad - No types
const fetchRooms = (id) => { ... };

// ✅ Good - Explicit types
interface FetchRoomsParams {
  id: string;
  includeAvailability?: boolean;
}

const fetchRooms = ({ id, includeAvailability = false }: FetchRoomsParams): Promise<Room[]> => {
  // ...
};
```

#### 2. Avoid `any` Type
```typescript
// ❌ Bad
const data: any = await fetch(...);

// ✅ Good
interface ApiResponse {
  rooms: Room[];
  total: number;
}
const data: ApiResponse = await fetch(...);

// ✅ Also acceptable when truly unknown
const data: unknown = await fetch(...);
if (isApiResponse(data)) {
  // Type guard narrows to ApiResponse
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HeyitSaif) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
