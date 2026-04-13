---
trigger: always_on
description: Build a platform for delivering online courses that includes:
---

# Development Guide for a Coursera-like Learning Platform

## Project Overview

Build a platform for delivering online courses that includes:

- A catalog of courses organized by categories
- Advanced search and filtering
- Instructor profiles and reviews
- Course content (video, documents, quizzes)
- Knowledge assessment system (quizzes and coding-style challenges)
- Certificate issuance for course completion
- Shopping cart and payment

## Technology Stack

- Frontend Framework: React 19.2.0 + TypeScript
- Build Tool: Vite
- Styling: TailwindCSS 3.3.6
- UI Components: shadcn/ui + Lucide React Icons
- Form Management: React Hook Form + Zod Validation
- Routing: React Router DOM 7.0.0
- Utilities: clsx, tailwind-merge, class-variance-authority

## Project Structure

```
src/
├── components/
│   ├── ui/                 # shadcn/ui components (Button, Card, Dialog, etc.)
│   ├── common/             # Layout components (Header, Footer, Navbar)
│   └── course/             # Course-specific components (CourseCard, Filters, etc.)
├── pages/                  # Page components (Home, CourseDetail, etc.)
├── types/                  # TypeScript type definitions
├── lib/                    # Utility functions, constants
├── hooks/                  # Custom React hooks
├── App.tsx                 # Main app component with routing
└── main.tsx                # App entry point
```

## Data Management & Mock Data Strategy

### Current Architecture

Currently, all frontend data is provided by mock data stored in the `src/mocks/` folder:

```
src/
├── mocks/
│   ├── courses.ts        # Mock course data
│   ├── instructors.ts    # Mock instructor data
│   ├── categories.ts     # Mock category data
│   └── index.ts          # Export all mock data
```

### Design Pattern for Easy Migration to a Real API

To make it straightforward to switch from mock data to server API calls in the future, follow this pattern:

#### 1. Data Service Layer (`src/services/`)

Create a service layer that centralizes data fetching logic and isolates components from where data comes from:

```typescript
// src/services/courseService.ts
export async function getCourses(): Promise<Course[]> {
  // Current: import from mock
  // const { courses } = await import('../mocks/courses');
  // return courses;
  // Future: API call
  // const response = await fetch('/api/courses');
  // return response.json();
}

export async function getCourseById(id: string): Promise<Course | undefined> {
  // Current: return courses.find(c => c.id === id);
  // Future: return fetch(`/api/courses/${id}`).then(r => r.json());
}

export async function getInstructors(): Promise<Instructor[]> {
  // ...
}
```

#### 2. Custom Hooks (`src/hooks/`)

Wrap service calls in custom hooks that provide loading and error states to components:

```typescript
// src/hooks/useCourses.ts
export function useCourses() {
  const [courses, setCourses] = useState<Course[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    setLoading(true);
    getCourses()
      .then(setCourses)
      .catch((err) => setError(err.message))
      .finally(() => setLoading(false));
  }, []);

  return { courses, loading, error };
}
```

#### 3. Component Usage

Components should call hooks and not import mock data directly:

```tsx
// Good: ✓
function CourseList() {
  const { courses, loading } = useCourses();
  return (
    <div>
      {courses.map((c) => (
        <CourseCard key={c.id} course={c} />
      ))}
    </div>
  );
}

// Avoid: ✗
// import { mockCourses } from '../mocks/courses';
// function CourseList() {
//   return <div>{mockCourses.map(...)}</div>;
// }
```

### Migration Checklist

When you're ready to move to a real API:

1. Create API endpoints that match the current data structure
2. Update the service layer (`src/services/*.ts`) to call the API instead of mocks
3. Add error handling and optional retry logic
4. Ensure hooks provide loading and empty states
5. Test the full data flow end-to-end
6. Keep or remove `src/mocks/` for development and testing as needed

### Best Practices

- Never import mock data directly in components — always go through the service layer
- Keep TypeScript types in `src/types/` aligned with API responses
- Centralize API configuration (create `src/config/api.ts` for base URL, headers, etc.)
- Surface errors from the service layer to components via hooks and use error boundaries where appropriate

## Design System & Colors (Inspired by Coursera)

- **Primary Blue**: #4f46e5 (Main CTA, navigation)
- **Dark Blue**: #1e1b4b (Text headings, dark mode)
- **Light Blue**: #f0f4ff (Backgrounds, hover states)

### Accent Colors

- **Secondary Purple**: #ec4899 (Highlights, featured courses)
- **Success Green**: #10b981 (Completion status, ratings)
- **Warning Orange**: #f59e0b (Limited slots, urgent)
- **Danger Red**: #ef4444 (Error states)

### Neutral Palette

- **White/Very Light**: #f9fafb (Primary backgrounds)
- **Light Gray**: #e5e7eb (Borders, subtle dividers)
- **Medium Gray**: #6b7280 (Secondary text, captions)
- **Dark Gray**: #1f2937 (Body text, content)
- **Very Dark**: #111827 (Headings, emphasis)

## Component Guidelines

### 1. Button Component


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PDK-23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
