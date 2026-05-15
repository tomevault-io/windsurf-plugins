---
trigger: always_on
description: This document contains recurring instructions, patterns, and best practices for maintaining the 3rd Party Tracer project.
---

# AI Agent Instructions for 3rd Party Tracer

This document contains recurring instructions, patterns, and best practices for maintaining the 3rd Party Tracer project.

## Core Principles

### 1. Client-Side Only Architecture
- **NEVER** add server-side code or dependencies
- All functionality must run entirely in the browser
- Use DNS over HTTPS (DoH) for DNS queries
- No backend APIs, no data collection, no external service dependencies
- Keep the tool privacy-focused and standalone

### 2. Excluded Directories
- Always work in the root project files only

### 3. File Organization
```
Root Project Files (MODIFY THESE):
├── index.html
├── about.html
├── css/style.css
├── js/
│   ├── logger.js
│   ├── app.js
│   ├── dns-analyzer.js
│   ├── service-detection-engine.js
│   ├── data-processor.js
│   ├── ui-renderer.js
│   ├── analysis-controller.js
│   ├── export-manager.js
│   └── theme-toggle.js
├── CHANGELOG.md
└── .github/workflows/deploy.yml

Do NOT Modify:
└── logs/ (historical data)
```

## Release Process

### When Creating a New Release

**Follow these steps in order:**

#### Step 1: Update CHANGELOG.md
1. Move all `[Unreleased]` changes to a new version section
2. Format: `## [X.Y.Z] - YYYY-MM-DD`
3. Use semantic versioning:
   - MAJOR: Breaking changes
   - MINOR: New features, backward compatible
   - PATCH: Bug fixes, optimizations
4. Keep an empty `[Unreleased]` section at the top

Example:
```markdown
## [Unreleased]

## [1.0.2] - 2025-10-27

### Added
- New feature here

### Fixed
- Bug fix here
```

#### Step 2: Update Cache Busting Versions

**Update version parameter in all these locations:**

**File: `index.html`**
- `<link rel="stylesheet" href="css/style.css?v=X.Y.Z">`
- `<script src="js/logger.js?v=X.Y.Z">`
- `<script src="js/export-manager.js?v=X.Y.Z">`
- `<script src="js/dns-analyzer.js?v=X.Y.Z">`
- `<script src="js/service-detection-engine.js?v=X.Y.Z">`
- `<script src="js/data-processor.js?v=X.Y.Z">`
- `<script src="js/ui-renderer.js?v=X.Y.Z">`
- `<script src="js/analysis-controller.js?v=X.Y.Z">`
- `<script src="js/app.js?v=X.Y.Z">`
- `<script src="js/theme-toggle.js?v=X.Y.Z">`

**File: `about.html`**
- `<link rel="stylesheet" href="css/style.css?v=X.Y.Z">`
- `<script src="js/theme-toggle.js?v=X.Y.Z">`

#### Step 3: Verify All Versions Updated
Run this check:
```bash
grep -r "?v=1\.0\." --include="*.html" .
```
All results should show the NEW version number.

#### Step 4: Update GitHub Actions (if JS files changed)
**File: `.github/workflows/deploy.yml`**

If you added or removed JavaScript files, update:
1. The file copy section (around line 75-85)
2. The verification section (around line 108-122)
3. The deployment summary file count (around line 182)

Example from v1.0.2:
```yaml
# JavaScript files
cp js/logger.js _site/js/
cp js/app.js _site/js/
# ... (8 JS files total, removed service-registry.js and subdomain-registry.js)
echo "✅ Copied 8 JavaScript files"
```

## Recurring Instructions & Patterns

### Code Optimization
When asked to optimize or clean up code:
1. Read through all relevant files systematically
2. Use `grep` to find function definitions and usage
3. Cross-reference to identify unused code
4. Check for duplicate functions
5. Look for dead code (functions that are never called)
6. Remove unused files and their references in HTML
7. Update deployment workflow if files are removed
8. Document all changes in CHANGELOG.md

**Common optimization targets:**
- Unused JavaScript classes/functions
- Duplicate function definitions
- Unused CSS selectors
- Redundant logic
- Console logging (implement conditional logging with debug mode)

### Security Reviews
When reviewing for security issues:
1. **Domain Confusion Vulnerabilities**: Use `isDomainOrSubdomain()` helper instead of `.includes()` for domain checks
2. **Input Validation**: Always validate and sanitize user inputs
3. **XSS Prevention**: Use proper DOM manipulation (avoid innerHTML with user data)
4. **CORS**: Ensure all external API calls respect CORS policies
5. **Client-Side Security**: No sensitive data storage, no API keys in code

**Pattern to fix domain confusion:**
```javascript
// ❌ VULNERABLE
if (target.includes('amazonaws.com')) { ... }

// ✅ SECURE
if (this.isDomainOrSubdomain(target, 'amazonaws.com')) { ... }
```

### Dark Mode Styling
When fixing dark mode issues:
1. Check for hardcoded colors (hex values instead of CSS variables)
2. Ensure both light and dark mode overrides exist
3. **Always test visual elements in both themes**
4. Common issues: white backgrounds in dark mode, poor contrast, dull text colors

**Standard color replacement patterns:**
```css
/* Replace hardcoded colors with CSS variables */
color: #495057;  →  color: var(--text-color);
color: #6c757d;  →  color: var(--text-secondary);
background: #f8f9fa;  →  background: var(--bg-tertiary);
border: 1px solid #e9ecef;  →  border: 1px solid var(--border-color);
background: white;  →  background: var(--card-bg);
```

**Dark mode pattern with light mode override:**
```css
.element {
    color: var(--text-color);  /* Uses theme variable (dark mode default) */
    background: rgba(255, 193, 7, 0.1);  /* Dark mode background */
}

[data-theme="light"] .element {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyfinoid/3ptracer](https://github.com/cyfinoid/3ptracer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
