---
trigger: always_on
description: This document provides instructions for Claude (AI assistant) when working on this MCP server project.
---

# Claude Instructions for YouTube Transcript MCP Server

This document provides instructions for Claude (AI assistant) when working on this MCP server project.

## Project Overview

This is an MCP (Model Context Protocol) server that provides YouTube video transcript retrieval capabilities. It must remain compatible with Claude Code, Claude Desktop, and all MCP-enabled systems.

## Critical Rules

### 1. Always Run Tests After Changes

**MANDATORY**: After making ANY changes to the codebase, you MUST run the test suite:

```bash
npm test
# OR
node test-server.js
```

**This is not optional.** The test suite validates:
- JSON Schema compliance (required for Claude API compatibility)
- MCP protocol correctness
- Functional behavior of all tools with actual YouTube transcript retrieval

### 2. Schema Compliance Requirements

When modifying or adding tool schemas in `index.js`, ALL schemas MUST include:

```javascript
inputSchema: {
  $schema: "https://json-schema.org/draft/2020-12/schema",  // REQUIRED!
  type: "object",
  properties: {
    // ... your properties
  },
  required: ["list", "of", "required", "fields"],
  additionalProperties: false  // STRONGLY RECOMMENDED
}
```

**Common mistakes to avoid:**
- ❌ Missing `$schema` field → causes Claude API errors
- ❌ Using wrong schema version (must be draft 2020-12, not draft-07)
- ❌ Using `default` keyword → not validated, document in description instead
- ❌ Invalid property types (use "integer" not "int", "string" not "str")
- ❌ Missing `additionalProperties: false` → allows unexpected parameters

### 3. Testing Workflow

When making changes, follow this workflow:

1. **Make your changes** to `index.js` or other files
2. **Run tests immediately**: `npm test`
3. **Verify compliance tests pass**:
   ```
   ✓ Schema compliance check PASSED for: [tool-name]
   ```
4. **Verify functional tests pass** (or fail gracefully if network issues occur)
5. **Only proceed** if ALL compliance tests pass

### 4. If Tests Fail

If compliance tests fail:

```
✗ Schema compliance check FAILED for: get-transcript
```

**Do NOT ignore this.** Fix the schema before proceeding:

1. Check the error message for what's missing
2. Refer to `TESTING.md` for examples
3. Fix the schema
4. Run tests again
5. Repeat until all tests pass

### 5. Code Changes Guidelines

#### Adding a New Tool

```javascript
{
  name: "new-tool",
  description: "Clear description of what this tool does",
  inputSchema: {
    $schema: "https://json-schema.org/draft/2020-12/schema",  // Don't forget!
    type: "object",
    properties: {
      param1: {
        type: "string",
        description: "Description with defaults documented here. Default: 'value'"
      }
    },
    required: ["param1"],
    additionalProperties: false
  }
}
```

**After adding:** Run `npm test` immediately.

#### Modifying Existing Tools

1. Make your changes
2. Run `npm test`
3. If compliance fails, you broke something - fix it
4. Document what changed in your commit

#### URL Parsing Logic

When modifying URL parsing, ensure support for ALL these formats:

```javascript
// Must support:
"https://www.youtube.com/watch?v=VIDEO_ID"
"https://youtu.be/VIDEO_ID"
"https://m.youtube.com/watch?v=VIDEO_ID"
"VIDEO_ID"  // Direct 11-character ID
```

Test each format after changes.

### 6. Testing Edge Cases

When adding new functionality, test:

- ✓ Valid YouTube URLs work correctly
- ✓ Invalid URLs fail gracefully with helpful errors
- ✓ Missing transcripts are handled properly
- ✓ Various URL formats are supported
- ✓ Schema validates correctly with Ajv

### 7. Documentation Requirements

When changing functionality:

1. Update `README.md` if user-facing changes
2. Update `TESTING.md` if testing procedures change
3. Add examples for new features
4. Document breaking changes clearly
5. Update `LEARNING.md` with any new insights

### 8. Git Commit Guidelines

Use clear, descriptive commit messages:

```bash
git commit -m "Add feature: [description]

- Specific change 1
- Specific change 2
- Run tests: all passing"
```

**Before committing:** Run `npm test` one final time.

### 9. Git Push and npm Publishing Protocol

**CRITICAL WORKFLOW - NEVER SKIP:**

#### After Any Git Commit:

1. **ALWAYS ask user before pushing to git**: "Would you like me to push these changes to git?"
2. **If user approves push**: Execute `git push`
3. **IMMEDIATELY after successful push**: ALWAYS ask "Would you like me to publish this to npm?"

**This is MANDATORY. Never skip asking about npm publish after a git push.**

#### Publishing to npm

**Manual publish:**

```bash
# Bump version
npm version patch  # or minor/major

# Run tests
npm test

# Publish
npm publish

# Push to git
git push && git push --tags
```

**When to publish:**
- After fixing bugs (patch)
- After adding new features (minor)
- After breaking changes (major)
- After significant improvements
- When user requests it

## Development Workflow Checklist

Use this checklist for every change:

- [ ] Made code changes
- [ ] Ran `npm test`
- [ ] All compliance tests passed
- [ ] Functional tests behave as expected
- [ ] Updated documentation if needed
- [ ] Committed changes with clear message
- [ ] **ASKED USER** before pushing to git

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hancengiz/youtube-transcript-mcp](https://github.com/hancengiz/youtube-transcript-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
