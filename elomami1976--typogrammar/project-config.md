---
trigger: always_on
description: TypoGrammar is a React + TypeScript educational web app for English grammar learning, built with Vite and deployed to static hosting (Hostinger). The app uses BrowserRouter for client-side routing and localStorage for progress tracking.
---

# TypoGrammar AI Coding Instructions

## Project Overview
TypoGrammar is a React + TypeScript educational web app for English grammar learning, built with Vite and deployed to static hosting (Hostinger). The app uses BrowserRouter for client-side routing and localStorage for progress tracking.

## Architecture

### Content-Driven Design
The app is primarily **content-centric**, not feature-heavy. Most grammar topics, quizzes, blog posts, and vocabulary data live in `src/constants/` as large TypeScript files with React.ReactNode content:
- `grammarTopics.tsx` - Main grammar lessons with JSX content (1700+ lines)
- `quizData.ts` - Quiz questions mapped to topics
- `blogPosts.tsx` - Blog content with embedded components
- `irregularVerbs.ts`, `phrasalVerbs.ts`, `idioms.ts`, `confusedWords.ts`, etc. - Reference data

When adding content, update the relevant constant file. Content uses custom article components from `src/components/ArticleComponents.tsx` (ArticleParagraph, ArticleHeading, InlineCode, CodeBlock, ExampleList, BulletList) and visual aids from `src/components/VisualAids.tsx` (TimelineDiagram, SentenceTransformationDiagram).

**Example content structure pattern:**
```tsx
// In grammarTopics.tsx
{
  id: 'topic-id',
  title: 'Topic Title',
  category: 'Category Name',
  content: (
    <>
      <ArticleParagraph>Introduction text...</ArticleParagraph>
      <ArticleHeading>Section Title</ArticleHeading>
      <ExampleList items={["Example 1", "Example 2"]} />
      <CodeBlock>{`Code or grammar pattern`}</CodeBlock>
    </>
  ),
}
```

### Routing & Code Splitting
- Uses BrowserRouter for clean URLs (requires server-side routing config)
- Entry point: `src/index.tsx` wraps App in `<BrowserRouter>` and context providers (LanguageProvider, ProgressProvider)
- All routes except HomePage are lazy-loaded in `src/App.tsx` with React.lazy()
- Vite config manually chunks react-vendor for optimal loading
- Route pattern: `/topics/:topicId` for grammar topics, direct paths for other pages
- **Server config**: `.htaccess` in `public/` redirects all routes to `index.html` for client-side routing (Apache mod_rewrite required)
- Alternative: `_redirects` file also present in `public/` for Netlify/similar platforms

### State Management
**No Redux or complex state libraries.** Three context-based state patterns:
1. **Global Progress**: `src/contexts/ProgressContext.tsx` provides `useProgress()` hook
   - Tracks completed topics and quiz scores
   - Persists to localStorage via `src/services/progressService.ts`
   - Only updates best scores (doesn't overwrite with lower scores)

2. **Language/i18n**: `src/contexts/LanguageContext.tsx` provides `useLanguage()` hook
   - Manages current language state (Language type from `src/i18n/translations.ts`)
   - Persists to localStorage with key 'language'
   - Sets document.documentElement `lang` and `dir` attributes (rtl for Arabic)
   - Provides `t` object with translations for current language
   - Use `const { language, setLanguage, t, dir } = useLanguage();` to access

3. **Local Component State**: useState for UI interactions (theme, mobile menu, quiz state)

### Styling Approach
- **Tailwind CSS v3.4.0 via npm** (installed as dev dependency, not CDN)
- Dark mode: class-based (`darkMode: 'class'` in tailwind.config.js), toggled via `dark:` prefix
- Dark mode state stored in localStorage and managed in `src/components/Layout.tsx`
- Responsive design with mobile-first breakpoints (md:, lg:)
- Custom fonts: Inter (body), Poppins (headings) loaded from Google Fonts with async optimization
- PostCSS with Tailwind and Autoprefixer for CSS processing
- CSS directives in `src/index.css`: `@tailwind base`, `@tailwind components`, `@tailwind utilities`

### Page Structure
All pages follow this pattern:
```tsx
import usePageMetadata from '../hooks/usePageMetadata';

const SomePage = () => {
  usePageMetadata({
    title: 'Page Title | TypoGrammar',
    description: 'SEO description'
  });
  // Page content
}
```

Layout hierarchy: Layout.tsx → Header + Sidebar + Outlet (page content) + Footer

### Google AdSense Integration
- Publisher ID set in `index.html` head
- Use `<GoogleAd adSlot="..." />` component from `src/components/GoogleAd.tsx`
- Component auto-pushes to adsbygoogle on mount
- Pass `adTest="on"` prop for local testing

## Development Workflow

### Commands
```bash
npm install          # Install dependencies
npm run dev          # Start dev server (usually http://localhost:5173)
npm run build        # Build for production → dist/ folder
npm run preview      # Preview production build locally
```

### Common Development Workflows

**Adding a new grammar topic with quiz:**
```bash
# 1. Add content to grammarTopics.tsx
# 2. Add quiz to quizData.ts with matching topicId
# 3. Test: npm run dev, navigate to /topics/your-topic-id
# 4. Verify quiz integration and progress tracking
```

**Adding a new standalone page:**
```bash
# 1. Create YourPage.tsx in src/pages/
# 2. Add lazy import in App.tsx: const YourPage = lazy(() => import('./pages/YourPage'));

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Elomami1976/typogrammar](https://github.com/Elomami1976/typogrammar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
