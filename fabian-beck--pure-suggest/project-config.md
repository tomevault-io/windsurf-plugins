---
trigger: always_on
description: PUREsuggest is a Vue 3 scientific literature search tool that suggests publications through citation-based recommendations. This is a single-page application built with Vue 3, Vite, Pinia stores, and D3.js network visualization.
---

# PUREsuggest - GitHub Copilot Instructions

PUREsuggest is a Vue 3 scientific literature search tool that suggests publications through citation-based recommendations. This is a single-page application built with Vue 3, Vite, Pinia stores, and D3.js network visualization.

**ALWAYS follow these instructions first and only fallback to additional search and context gathering if the information here is incomplete or found to be in error.**

## Bootstrap and Setup

Always run these commands in sequence when working with a fresh clone:

```bash
# Install dependencies (NEVER CANCEL - Network restrictions require skip flag)
PUPPETEER_SKIP_DOWNLOAD=true npm install
# Takes ~6 seconds. Use timeout of 300+ seconds for safety.

# Verify linting passes
npm run lint
# Takes ~1.5 seconds. Should complete with no errors.

# Build production version (NEVER CANCEL)
npm run build
# Takes ~6 seconds. Set timeout to 300+ seconds. Creates dist/ directory.
```

## Testing Commands

**CRITICAL**: Set appropriate timeouts for ALL test commands. NEVER CANCEL builds or tests.

```bash
# Run unit tests (NEVER CANCEL)
npm test
# Takes ~14 seconds. Runs 436 tests across 36 test files. Set timeout to 300+ seconds.

# Run performance tests (NEVER CANCEL)
npm run test:perf
# Takes ~8 seconds. Browser tests may skip due to missing Chrome - this is expected in sandboxed environments. Set timeout to 300+ seconds.

# Run all tests (NEVER CANCEL)
npm run test:all
# Takes ~22 seconds total. Combines unit and performance tests. Set timeout to 600+ seconds.
```

## Development Server

```bash
# Start development server
npm run dev
# Starts on port 8080, auto-retries on 8081, 8082 if busy.
# Access at http://localhost:8080 (or auto-assigned port)

# Preview production build
npm run preview
# Serves production build on port 4173
# Access at http://localhost:4173
```

## Validation Scenarios

After making changes, ALWAYS test these core user workflows:

### Basic Literature Search Workflow

1. **Start application**: `npm run dev` and access web interface
2. **Add publication by DOI**: Test DOI input field (e.g., "10.1109/TVCG.2024.3456199")
3. **Verify suggestions appear**: Check that citation-based suggestions are generated
4. **Test keyword boosting**: Add boost keywords and verify ranking changes
5. **Apply filters**: Use publication year, author, or citation count filters
6. **Export functionality**: Test BibTeX export and session save/load

### Performance Validation

1. **Load testing**: Add multiple publications (10+) and verify responsiveness
2. **Network visualization**: Ensure D3.js citation network renders without errors
3. **Memory usage**: Monitor for memory leaks during extended use

### Component Integration Testing

1. **Selected Publications**: Verify publications appear in selection list
2. **Suggested Publications**: Check ranking and metadata display
3. **Author details**: Test author modal and co-author information
4. **Filter menu**: Verify all filter options work correctly

## Key Project Structure

### Core Business Logic (`src/core/`)

- `Publication.js` - Publication model with metadata fetching and scoring
- `Author.js` - Author aggregation and scoring algorithms
- `Filter.js` - Publication filtering logic
- `PublicationSearch.js` - DOI-based search functionality

### Main Components (`src/components/`)

- `App.vue` - Main application layout with responsive grid
- `NetworkVisComponent.vue` - D3-powered citation network visualization
- `SelectedPublicationsComponent.vue` - User's selected publications list
- `SuggestedPublicationsComponent.vue` - Ranked publication suggestions
- `PublicationComponent.vue` - Individual publication display
- `FilterMenuComponent.vue` - Advanced filtering interface

### State Management (`src/stores/`)

- `session.js` - Selected publications, suggestions, filters, boost keywords
- `interface.js` - UI state, modals, loading states, mobile responsiveness
- `queue.js` - Temporary queues for batch operations
- `author.js` - Author aggregation and scoring

### Services (`src/services/`)

- `SuggestionService.js` - Dedicated service for computing publication suggestions

### Testing (`tests/`)

- `tests/unit/` - Component and utility unit tests (36 files, 436 tests)
- `tests/performance/` - Load and rendering benchmarks

## Common Build Issues and Solutions

### Puppeteer Installation Failure

```bash
# Error: Failed to set up chrome-headless-shell
# Solution: Always use skip flag
PUPPETEER_SKIP_DOWNLOAD=true npm install
```

### Port Conflicts

- Development server auto-retries ports 8080 → 8081 → 8082
- Preview server uses port 4173
- Check console output for actual assigned port

### Performance Test Browser Failures

- Browser tests skip when Chrome unavailable (expected in sandboxed environments)
- Performance scale tests still run and provide valuable metrics
- This is normal and not a blocker

## Development Best Practices

### Code Style and Testing

- ALWAYS run `npm run lint` before committing changes
- Run `npm test` to ensure unit tests pass (TDD approach recommended)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fabian-beck/pure-suggest](https://github.com/fabian-beck/pure-suggest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
