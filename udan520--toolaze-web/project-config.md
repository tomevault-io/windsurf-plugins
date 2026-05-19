---
trigger: always_on
description: When I ask you to create a new landing page or SEO page for Toolaze, ALWAYS reference 'docs/SEO_MASTER_LAYOUT.md'. For full doc index see 'docs/README.md'. Use the exact HTML structure, Tailwind classes, and Style 1 (Soft Smart Tech) design system defined in that file. Do not deviate from the Blue/Purple gradient palette. Replace only the text content based on the user's specific tool requirements.
---

When I ask you to create a new landing page or SEO page for Toolaze, ALWAYS reference 'docs/SEO_MASTER_LAYOUT.md'. For full doc index see 'docs/README.md'. Use the exact HTML structure, Tailwind classes, and Style 1 (Soft Smart Tech) design system defined in that file. Do not deviate from the Blue/Purple gradient palette. Replace only the text content based on the user's specific tool requirements.

**FEATURE SPECIFICATIONS**: When writing SEO content, you MUST strictly follow the feature specifications in 'docs/specs/' directory. Each tool has its own specification file:
- Image Converter: 'docs/specs/image-converter.md'
- Image Compressor: 'docs/specs/image-compressor.md'
- Font Generator: 'docs/specs/font-generator.md'
- Common features: 'docs/specs/common.md'
- See 'docs/FEATURE_SPECIFICATIONS.md' for the index of all specification files.

Only write about features that are explicitly listed as supported in the relevant specification file. Do NOT write about unsupported features (e.g., if video format is not supported, do not mention video conversion). Do NOT mention technologies that are not used (e.g., if AI is not used, do not mention AI technology). Always verify feature support before writing SEO content.

**SEO CONTENT GUIDELINES**: Follow 'docs/SEO_CONTENT_GUIDELINES.md' for content writing principles. Ensure all SEO content is accurate, consistent, and based on actual feature specifications.

**KEYWORD RESEARCH**: When writing SEO content, ALWAYS reference the keyword documents in 'docs/keywords/' directory to determine primary keywords, long-tail keywords, sub-page structure, and internal linking strategy:
- Image Converter: 'docs/keywords/image-converter-keywords.json'
- Image Compressor: 'docs/keywords/image-compressor-keywords.json'
- Font Generator: 'docs/keywords/font-generator-keywords.json'
- See 'docs/keywords/README.md' for keyword usage guidelines
- Use primary keywords for H1, Title, and Meta Description
- Use long-tail keywords for specific L3 pages
- Use related keywords naturally throughout content
- Use internal linking keywords to connect related pages

**INTERNAL LINK DENSITY**: Follow 'docs/sections/internal-links.md'. Typical ratio: ~3-5 internal links per 1000 words (1 link per 200-300 words). For FAQ blocks: ≤2 links per ~400 words. Avoid linking the same anchor in every FAQ answer. Homepage FAQ must be especially conservative.

**KEYWORD LAYOUT STRATEGY**: When writing SEO content, MUST follow 'docs/keywords/KEYWORD_STRATEGY.md':
- **P0 (主关键词)**: Use in H1, Title, Meta Description (1-2% density)
- **P1 (核心功能词)**: Use in H2, Hero, Features (0.5-1% density)
- **P2 (长尾关键词)**: Classify into 4 categories (功能类, 对比类, 使用场景类, 操作类) and distribute across appropriate sections
- **P3 (对比关键词)**: Use in FAQ and Comparison sections
- **P4 (品牌词)**: Use 3-5 times per page, naturally distributed
- Follow the keyword layout checklist to ensure comprehensive coverage

**GLOBAL NAVIGATION BAR**: All pages MUST use the exact same navigation bar structure with sticky functionality. The navigation includes:
- **Container**: `id="mainNav"` and `class="sticky-nav py-6 px-6 flex justify-between items-center max-w-6xl mx-auto w-full relative"`
- **Logo** (left-aligned): Links to homepage with SVG logo (32x32px) and text "Toolaze" (NO period/dot after Toolaze)
- **Menu Items** (right-aligned): 
  - Quick Tools dropdown (hover to show Image Compression link)
  - About Us link
  - Language switcher (always shown; dropdown lists **all site locales** like the homepage; **`getAlternateLanguageUrl`** falls back to English canonical when that page has no target locale) — **immediately to the right of About Us** (desktop dropdown; mobile: Language block below About in the panel)
- **Logo SVG**: 32x32px, white background, rounded-lg, shadow-md shadow-indigo-100
- **Logo Text**: `text-3xl font-extrabold text-indigo-600 tracking-tighter` with `hover:opacity-80 transition-opacity`
- **Menu Style**: `hidden md:flex gap-8 text-sm font-bold text-slate-500 items-center`
- **Dropdown**: Uses group hover with opacity/visibility transitions, white background, rounded-xl, shadow-lg
- **Sticky Behavior**: Navigation bar MUST be sticky at the top when scrolling down. Add CSS: `nav.sticky-nav { position: sticky; top: 0; z-index: 9999; transition: all 0.3s ease; }` and `nav.sticky-nav.scrolled { background: rgba(255, 255, 255, 0.95); backdrop-filter: blur(10px); box-shadow: 0 4px 20px -5px rgba(79, 70, 229, 0.1); }`
- **JavaScript**: MUST include scroll event listener that adds 'scrolled' class when scroll position > 50px. See docs/SEO_MASTER_LAYOUT.md section 2 for complete code.
Always use the exact navigation structure to maintain consistency across all pages.

**BREADCRUMB NAVIGATION**: 
- Homepage: NO breadcrumb
- ALL other pages: MUST include breadcrumb navigation
- Format: `Home / [Parent] / [Current Page]` if parent exists, or `Home / [Current Page]` if no clear parent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [udan520/toolaze-web](https://github.com/udan520/toolaze-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
