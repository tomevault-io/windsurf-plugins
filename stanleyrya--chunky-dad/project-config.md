---
trigger: always_on
description: Static website for gay bear travel guide with city guides, events, and bear-owned businesses. Built with vanilla HTML, CSS, and JavaScript.
---

# chunky.dad Website - Cursor Rules

## Project Overview
Static website for gay bear travel guide with city guides, events, and bear-owned businesses. Built with vanilla HTML, CSS, and JavaScript.

## Key Guidelines
- **NEVER create new markdown files** - absolutely no *.md files except existing README
- **No documentation files** - do not create summaries, guides, or documentation
- **Free tools only** - no API keys required
- **GitHub Pages hosting** - deployed via GitHub Pages

## Code Style
- Use semantic HTML5 elements
- CSS Grid/Flexbox for layouts
- ES6+ JavaScript (const/let, arrow functions, template literals)
- 2-space indentation
- Vanilla JavaScript preferred over libraries

## File Structure
```
/
├── index.html          # Main landing page
├── city.html           # City guide template
├── styles.css          # Main stylesheet
├── js/                 # Modular JavaScript architecture
│   ├── logger.js       # Centralized logging system
│   ├── city-config.js  # City configuration data
│   ├── navigation.js   # Navigation and scrolling
│   ├── page-effects.js # Animations and visual effects
│   ├── forms.js        # Form handling and validation
│   ├── calendar-core.js # Calendar data parsing and management
│   ├── dynamic-calendar-loader.js # Calendar UI and interactions
│   └── app.js          # Main application coordinator
├── data/               # JSON data files
├── scripts/            # Scriptable automation scripts (development)
│   ├── README.md       # Scripts documentation
│   ├── scriptable-complete-api.md # Complete Scriptable API reference (54 classes)
│   ├── bear-event-scraper-unified.js # Unified bear event scraper
│   └── scraper-config.json        # Scraper configuration
├── chunky-dad/         # Organized development files
│   ├── adapters/       # Environment-specific implementations
│   ├── parsers/        # Venue-specific parsing logic
│   └── shared-core.js  # Pure JavaScript business logic
├── adapters/           # Scriptable-compatible adapters (root level)
├── parsers/            # Scriptable-compatible parsers (root level)
├── bear-event-scraper-unified.js # Main scriptable file (root level)
├── display-saved-run.js # Past runs viewer (root level)
├── shared-core.js      # Core logic (root level)
├── scraper-input.json  # Configuration (root level)
├── testing/            # Test files and utilities
│   ├── index.html      # Dynamic test file browser
│   ├── manifest.json   # Auto-generated list of test files
│   └── generate-manifest.js # Script to update manifest
└── README.md
```

**SCRIPTABLE FILE ORGANIZATION**: 
- **Development**: ALWAYS edit files in the `scripts/` directory - this is the source of truth
- **NEVER create or edit root-level Scriptable files directly** - they are not used
- **Git repository**: Only the `scripts/` versions should be committed to Git
- The `chunky-dad/` folder contains organized backup copies for reference
- **For Scriptable app**: Use files directly from `scripts/` directory - no copying needed

## Testing Directory
- **Auto-updating manifest**: When adding new HTML files to testing/, run `npm run update-test-manifest`
- **Git hooks**: Run `npm run setup-hooks` to enable automatic manifest updates on commit
- **GitHub Actions**: Manifest is automatically updated when pushing changes to test files

## Current Patterns
- Modular JavaScript architecture with clear separation of concerns
- ES6 classes with inheritance (CalendarCore → DynamicCalendarLoader)
- Centralized application coordination via app.js
- CSS Grid for card layouts
- URL parameters for city guide content switching
- Dynamic calendar loading
- Responsive hamburger menu
- CSS custom properties for theming

## JavaScript Architecture
- **app.js**: Main coordinator, initializes all modules based on page type
- **navigation.js**: Handles mobile menu, smooth scrolling, navigation interactions
- **page-effects.js**: Manages animations, scroll effects, visual enhancements
- **forms.js**: Form validation, submission, user input processing
- **calendar-core.js**: Core calendar data parsing, iCal processing, date utilities
- **dynamic-calendar-loader.js**: Calendar UI rendering, interactions, map integration
- **js/app.js**: Main coordinator, initializes all modules based on page type

## Development
- **ALWAYS pull from main before making any changes** - run `git pull origin main` first
- Local server: `npm run dev` (Python HTTP server)
- Test responsive design on mobile
- Use browser dev tools for debugging
- **Console logging**: Always check F12 → Console for color-coded debug information
- **Performance monitoring**: Use logger timing functions for bottleneck identification
- **Error tracking**: All errors automatically logged with context and stack traces

## Error Handling & System Design
- **Graceful error handling**: Display user-friendly error messages when operations fail
- **NO complex legacy systems**: Avoid creating fallback systems, dual code paths, or complex compatibility layers
- **Fail fast**: If a feature doesn't work, show an error message rather than falling back to old systems
- **Simple is better**: Prefer simple, direct implementations over complex multi-path systems

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stanleyrya) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
