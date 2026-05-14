---
trigger: always_on
description: Documentation standards and when to update docs
---


# Documentation Standards

## Documentation Structure

### Essential Documents
1. **README.md** - User-facing overview, quick start, features
2. **PROJECT-OVERVIEW.md** - Architecture, concepts, data models
3. **CURRENT-STATE.md** - Implementation status, setup instructions
4. **DEVELOPER-GUIDE.md** - Quick reference for developers
5. **DESIGN.md** - Complete technical specification

### When to Create New Docs
Create new documentation when:
- Adding a major feature (e.g., MULTI-FILE-SUBMISSION-GUIDE.md)
- Implementing complex workflow (e.g., TESTING-GUIDE.md)
- Consolidating information (e.g., IMPLEMENTATION-SUMMARY.md)

Don't create docs for:
- Minor bug fixes
- Small UI tweaks
- Internal refactoring

## Update Checklist

### When Adding Features
- [ ] Update README.md with feature description
- [ ] Update CURRENT-STATE.md with implementation status
- [ ] Add to DEVELOPER-GUIDE.md if it's a new pattern
- [ ] Create feature-specific guide if complex
- [ ] Update relevant section in DESIGN.md

### When Changing Architecture
- [ ] Update PROJECT-OVERVIEW.md
- [ ] Update DESIGN.md
- [ ] Update any affected diagrams
- [ ] Note in CURRENT-STATE.md

### When Fixing Bugs
- [ ] If significant, note in CURRENT-STATE.md
- [ ] Update TROUBLESHOOTING section if applicable
- [ ] No need to update for minor fixes

## Document Formatting

### Headers
```markdown
# Main Title (H1 - only one per document)

## Section (H2)

### Subsection (H3)

#### Minor Section (H4)
```

### Code Blocks
Always specify language:
````markdown
```javascript
const example = "code";
```

```bash
npm install
```

```json
{
  "key": "value"
}
```
````

### File References
Use absolute paths from repo root:
```markdown
See [archiveGenerator.js](example-bounty-program/server/utils/archiveGenerator.js)
```

### Status Indicators
Use emoji for quick status:
```markdown
✅ Complete - Feature is done and tested
🔄 In Progress - Currently working on
⏳ Pending - Planned but not started
❌ Blocked - Cannot proceed
⚠️ Needs Attention - Issues to resolve
```

### Command Examples
Show full commands with context:
```markdown
### Start Backend
```bash
cd example-bounty-program/server
npm install
npm run dev
```
```

### Tables
Use for structured data:
```markdown
| Feature | Status | Notes |
|---------|--------|-------|
| Jobs    | ✅     | Complete |
| Submit  | ✅     | Complete |
```

## README.md Structure

```markdown
# Project Name

**Status:** 🟡 Status  
**Version:** 0.2.0

## Overview
Brief description (2-3 paragraphs)

## Quick Links
Links to essential docs

## Key Features
- Feature 1
- Feature 2

## Getting Started
Prerequisites and setup

## Development Roadmap
Phases with checkboxes

## FAQ
Common questions

## Support & Contact
Links and resources
```

## Technical Documentation

### Include
1. **Purpose** - Why this exists
2. **How it works** - Technical details
3. **Examples** - Code snippets
4. **Validation** - How to verify
5. **Troubleshooting** - Common issues

### Structure
```markdown
# Feature Name

**Status:** Complete  
**Date:** October 2025

## Overview
What and why

## Implementation Details
How it works

## Usage
Examples

## Testing
How to test

## Troubleshooting
Common issues
```

## API Documentation

### Endpoint Documentation
```markdown
### POST /api/endpoint

**Description:** What it does

**Request:**
```json
{
  "field": "value"
}
```

**Response:**
```json
{
  "success": true,
  "data": { }
}
```

**Errors:**
- 400: Invalid input
- 404: Not found
- 500: Server error
```

## Inline Code Documentation

### Function Documentation
```javascript
/**
 * Brief description of what function does
 * 
 * @param {Object} options - Configuration options
 * @param {string} options.field1 - Description of field1
 * @param {Array} options.field2 - Description of field2
 * @returns {Promise<Object>} - What it returns
 * @throws {Error} - When it might throw
 */
async function exampleFunction(options) {
  // Implementation
}
```

### Complex Logic
Add comments for:
- Non-obvious logic
- Business rules
- Important constraints
- Workarounds

```javascript
// Word count validation: max 200 words per specification
const wordCount = text.trim().split(/\s+/).length;
if (wordCount > 200) {
  return error;
}
```

## Version History

### In README.md
Track major changes:
```markdown
## Changelog

### v0.2.0 (October 2025)
- Added multi-file submission support
- Added custom submission narratives
- Enhanced CID display dialog

### v0.1.0 (September 2025)
- Initial implementation
- Basic job creation and submission
```

## Links and References

### External Links
Include context and target:
```markdown
See the [Verdikta Manifest Specification](https://docs.verdikta.com/verdikta-common/MANIFEST_SPECIFICATION/) for details.
```

### Internal Links
Use relative paths:
```markdown
See [DEVELOPER-GUIDE.md](DEVELOPER-GUIDE.md) for setup.
```

## Documentation Checklist

Before committing documentation changes:
- [ ] Spell check completed
- [ ] Code examples tested
- [ ] Links verified (no 404s)
- [ ] Status indicators accurate
- [ ] Version numbers updated
- [ ] Breaking changes highlighted
- [ ] Examples are complete
- [ ] File paths are correct
- [ ] Formatting is consistent

## Best Practices

1. **Keep it current** - Update docs with code changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [verdikta/verdikta-applications](https://github.com/verdikta/verdikta-applications) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
