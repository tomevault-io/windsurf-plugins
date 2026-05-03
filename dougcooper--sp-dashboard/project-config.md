---
trigger: always_on
description: This is a plugin for [Super Productivity](https://super-productivity.com) that provides a visual dashboard with metrics, charts, and daily summaries of your tasks and tracked time. The plugin is built with vanilla JavaScript, HTML, and CSS, and integrates with Super Productivity's plugin API.
---

# GitHub Copilot Instructions for Dashboard Plugin

## Project Overview

This is a plugin for [Super Productivity](https://super-productivity.com) that provides a visual dashboard with metrics, charts, and daily summaries of your tasks and tracked time. The plugin is built with vanilla JavaScript, HTML, and CSS, and integrates with Super Productivity's plugin API.

## Repository Structure

- `sp-dashboard/` - Main plugin files
  - `index.html` - Main UI with embedded JavaScript and CSS
  - `plugin.js` - Plugin registration and header button
  - `manifest.json.template` - Plugin metadata template
  - `icon.svg` - Plugin icon
- `tests/` - Vitest unit tests using JSDOM
- `Makefile` - Build and release automation
- `package.json` - Project dependencies and scripts

## Code Style and Standards

### JavaScript
- Use vanilla JavaScript (no frameworks) for plugin code
- Use modern ES6+ features (arrow functions, const/let, template literals)
- Follow camelCase naming convention for variables and functions
- Use descriptive function and variable names
- Keep functions focused and single-purpose
- Avoid global pollution - scope variables appropriately

### HTML/CSS
- All plugin code is embedded in `sp-dashboard/index.html`
- Use CSS custom properties (variables) for theming
- Support both light and dark themes via `.dark-theme` body class
- Use semantic HTML elements where possible
- Keep CSS organized by section with clear comments

### Comments
- Add comments for complex logic or non-obvious behavior
- Document functions with clear parameter and return descriptions
- Include examples in comments where helpful
- Don't over-comment obvious code

## Testing Requirements

### Test Framework
- Use Vitest for all tests
- Tests use JSDOM to load and test the actual HTML file
- Mock the PluginAPI for integration tests

### Test Coverage
- Write unit tests for all new features
- Test both success and error scenarios
- Include edge cases (empty results, invalid dates, etc.)
- Mock external dependencies (PluginAPI, clipboard, etc.)
- Maintain existing test structure and patterns

### Running Tests
```bash
npm test                 # Run tests once
npm run test:watch       # Watch mode
npm run test:coverage    # Generate coverage report
make test                # Alternative using Makefile
```

## Plugin API Integration

### PluginAPI Methods
- `PluginAPI.getTasks()` - Get active tasks
- `PluginAPI.getArchivedTasks()` - Get archived tasks
- `PluginAPI.getAllProjects()` - Get project information
- `PluginAPI.showSnack()` - Show notifications
- `PluginAPI.getStorage()` - Get persistent storage
- `PluginAPI.setStorage()` - Save persistent data

### Error Handling
- Always wrap PluginAPI calls in try-catch blocks
- Show user-friendly error messages via `showSnack`
- Log detailed errors to console for debugging
- Gracefully degrade when API calls fail

## Build and Release Process

### Building
```bash
make build               # Build plugin zip file
make clean               # Clean generated files
make help                # Show available commands
```

### Release Process
1. Update version in `package.json`
2. Run `make release-check` to verify prerequisites
3. Run `make release` to:
   - Generate manifest.json from template
   - Create plugin zip file
   - Create and push git tag
   - Create GitHub release
4. Follow [Semantic Versioning](https://semver.org/)

### Prerequisites for Releases
- GitHub CLI (`gh`) installed and authenticated
- Clean working directory (no uncommitted changes)
- Write access to repository

## Report Generation Guidelines

### Date Handling
- Use consistent date formatting: `YYYY-MM-DD` for storage
- Display dates in human-readable format: "Monday, January 15, 2024"
- Handle timezone considerations appropriately
- Validate date ranges (start must be <= end)

### Report Formatting
- Generate Markdown format for easy copy/paste
- Support two grouping modes: by Date and by Project
- Include time spent in human-readable format (e.g., "2h", "45m")
- Mark incomplete tasks with "WIP" indicator
- Show project names in brackets when applicable
- Include optional task notes when enabled

### Performance
- Filter tasks efficiently to avoid processing unnecessary data
- Use Sets to track processed tasks and avoid duplicates
- Minimize DOM manipulations
- Batch updates when possible

## UI/UX Guidelines

### Theme Support
- Detect theme from body class: `body.dark-theme`
- Use CSS custom properties for all colors
- Ensure good contrast in both light and dark themes
- Test all UI states in both themes

### User Feedback
- Show loading states for async operations
- Provide clear success/error messages
- Use appropriate icons in notifications (`ico` parameter)
- Validate user input before processing

### Accessibility
- Use semantic HTML elements
- Include appropriate ARIA labels where needed
- Ensure keyboard navigation works
- Maintain focus management in modals

## Data Persistence

### Storage
- Use `PluginAPI.getStorage()` and `setStorage()` for persistence

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dougcooper/sp-dashboard](https://github.com/dougcooper/sp-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
