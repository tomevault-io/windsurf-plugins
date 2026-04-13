---
trigger: always_on
description: Next.js 14/15 agency website showcasing digital solutions, services, and portfolio.
---

# UAE Digital Solution Agency - Next.js Frontend Rules

## Project Overview
Next.js 14/15 agency website showcasing digital solutions, services, and portfolio.
Frontend-only project connecting to headless WordPress backend via GraphQL.

## Tech Stack
- Next.js 14/15 (App Router)
- React 18+ with TypeScript
- Tailwind CSS 4.x
- Framer Motion
- React Hook Form + Zod
- WPGraphQL client

## Coding Standards

### Next.js App Router
- Server Components by default
- 'use client' only for: hooks, events, browser APIs, animations
- async/await for data fetching in Server Components
- generateMetadata() for all SEO
- generateStaticParams() for dynamic routes
- Route groups for organization: (marketing), etc.

### Component Patterns
```typescript
// Server Component (default pattern)
export async function CasesGrid() {
  const cases = await getCases()
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
      {cases.map(c => <CaseCard key={c.id} {...c} />)}
    </div>
  )
}

// Client Component (when needed)
'use client'
import { motion } from 'framer-motion'

export function AnimatedSection({ children }) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 24 }}
      whileInView={{ opacity: 1, y: 0 }}
      viewport={{ once: true }}
    >
      {children}
    </motion.div>
  )
}
```

### TypeScript
- Strict mode enabled
- No 'any' without justification
- Interface for props, Type for unions/intersections
- Explicit return types for exported functions
```typescript
interface CaseCardProps {
  title: string
  client: string
  image: string
  tags: string[]
  href: string
}

export function CaseCard({ title, client, image, tags, href }: CaseCardProps): JSX.Element {
  // Implementation
}
```

### Tailwind CSS
- Mobile-first responsive design
- Semantic class grouping: layout → spacing → colors → typography → effects
- Use cn() utility for conditional classes
- Extract variants with CVA for complex components
```typescript
import { cva } from 'class-variance-authority'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-lg transition',
  {
    variants: {
      variant: {
        primary: 'bg-blue-600 text-white hover:bg-blue-700',
        secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300',
      },
      size: {
        sm: 'h-9 px-3 text-sm',
        md: 'h-11 px-5',
        lg: 'h-13 px-7 text-lg',
      },
    },
  }
)
```

### Styling System
- Colors: Primary brand blue, neutral grays, semantic colors
- Spacing: Tailwind's 4px scale
- Typography: text-base (16px) body, text-6xl+ for heroes
- Border radius: rounded-lg default, rounded-2xl for cards
- Container: max-w-7xl mx-auto px-6 lg:px-8

### Image Optimization
- ALWAYS use next/image, never <img>
- priority={true} for above-fold images
- fill with object-cover for backgrounds
- Specify width/height for layout shift prevention
```typescript
<Image
  src={image}
  alt={title}
  width={1200}
  height={800}
  priority={isAboveFold}
  className="rounded-lg"
/>
```

### Animation Standards
- Framer Motion for all animations
- useReducedMotion support required
- Entrance animations: fade + subtle translate (y: 24)
- Duration: 0.3-0.6s for most animations
- viewport={{ once: true }} for scroll triggers
```typescript
<motion.div
  initial={{ opacity: 0, y: 24 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true, margin: '-100px' }}
  transition={{ duration: 0.5 }}
>
  {children}
</motion.div>
```

### Form Handling
- React Hook Form for state management
- Zod for schema validation
- Server Actions for submission
```typescript
const formSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  message: z.string().min(10),
})

type FormData = z.infer<typeof formSchema>

export function ContactForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(formSchema)
  })
  // Implementation
}
```

### Data Fetching (WordPress)
- Server Components fetch directly
- ISR with revalidate: 3600 for content pages
- Cache WordPress GraphQL responses
```typescript
export async function getCases() {
  const res = await fetch(process.env.WORDPRESS_GRAPHQL_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ query: CASES_QUERY }),
    next: { revalidate: 3600 }
  })
  return res.json()
}
```

### SEO Implementation
- Metadata API for all pages
- Structured data (JSON-LD)
- sitemap.ts and robots.ts
- Open Graph and Twitter cards
```typescript
export const metadata: Metadata = {
  title: 'Page Title - UAE Digital Solution',
  description: 'Compelling description for SEO',
  openGraph: {
    title: 'OG Title',
    description: 'OG Description',
    images: ['/og-image.jpg'],
  },
}
```

### Performance Rules
- Keep bundle size minimal
- Dynamic imports for heavy components
- Lazy load below-fold content
- Optimize images (WebP, proper sizing)
- Minimize 'use client' boundaries
- ISR for static-ish content
- Monitor Core Web Vitals

### Accessibility (WCAG 2.1 AA)
- Semantic HTML (nav, main, article, section)
- Proper heading hierarchy
- Alt text for images
- ARIA labels where semantic HTML insufficient
- Keyboard navigation
- Focus visible states

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/liszo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
