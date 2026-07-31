---
trigger: always_on
description: Follow these detailed instructions when processing Want submissions for the Web We Want project.
---

# Want Processing Instructions for GitHub Copilot

Follow these detailed instructions when processing Want submissions for the Web We Want project.

> **⚠️ CRITICAL RULE: A pull request created during wants processing must contain only one change — the addition of a new Markdown file under `wants/`. Modifying any other file (issue bodies, existing want files, workflows, configuration, or any other repository file) is strictly prohibited.**

## Quick Reference

### Required Markdown Fields

- `title`: "I want [description]" format
- `date`: ISO date string (current date/time)
- `submitter`: Submitter's name or "Anonymous"
- `number`: Submission ID from issue
- `tags`: Array of relevant technology labels
- `discussion`: GitHub issues URL pointing to the source issue as a placeholder (e.g. `https://github.com/WebWeWant/webwewant.fyi/issues/<issue-number>`). This will be updated to the discussion URL after a maintainer converts the issue to a discussion.
- `status`: "discussing"

### Optional Fields

- `related`: Array of related specifications/articles with title, url, type

### Essential Commands

- `npm run create-want -- <issue-number>` - Generate UUID and markdown template with discussion URL pre-populated (e.g. `npm run create-want -- 850`)
- `npm run check-duplicate "Want Title"` - Check for potential duplicates (fuzzy matching)
- `npm run validate-want wants/<ID>.md` - Validate markdown file
- `npm run convert-to-discussion -- <issue-number> [category-name]` - Convert a GitHub issue to a discussion via the GraphQL API and print the new discussion URL (e.g. `npm run convert-to-discussion -- 850`)

### Triage Decision Tree

1. **Spam/abuse/honeypot triggered?** → DELETE and close issue immediately
2. **Links to commercial services (>2 links)?** → FLAG as potential spam, review carefully
3. **Off-topic (not web platform)?** → REJECT with "off-topic" label, close issue
4. **Missing required fields?** → REJECT with explanation, close issue
5. **Potential duplicate?** → Run `npm run check-duplicate`, flag for human review if similar
6. **Passes all checks?** → CREATE markdown file and PR per "Creating Approved Submissions" section

### Similarity Score Guidelines (for duplicate checking)

- **90-100%** = Very likely duplicate → Flag for human review
- **70-89%** = Possibly duplicate → Flag for human review
- **<70%** = Probably unique → Proceed with processing

## 🎯 Mission & Scope

Web We Want focuses on **web platform evolution** including:

- HTML elements and attributes
- CSS properties and features
- JavaScript APIs and language features
- Browser behavior and standards
- Developer tools improvements
- Web accessibility enhancements

## 📝 Processing Workflow

### Step 1: Spam Detection ⚠️

**Immediately delete and close submissions that:**

1. **Triggered honeypot** - "website" field is filled (automated spam)
2. **Contain excessive links** - More than 2 external links (likely promotional)
3. **Commercial promotion** - Advertising services, products, or companies
4. **Abusive content** - Hateful, threatening, or inappropriate language
5. **Non-English content** - Unless directly related to internationalization features
6. **Obvious bot submissions** - Generic text, random characters, or template spam

**When deleting spam:**

- Close the issue immediately
- Add comment: "This submission was automatically detected as spam and removed."
- Do not engage with or provide detailed feedback on spam submissions

**Note:** Since GitHub Copilot cannot delete issues entirely, editing the content to remove harmful links and material is essential for security and preventing abuse.

### Step 2: Relevance Check 🎯

**Approve submissions that:**

1. **Relate to web platform evolution** - HTML, CSS, JavaScript, browser APIs, developer tools
2. **Include required information** - Clear title, detailed description, contact info
3. **Are technically feasible** - Not requesting impossible or deprecated features
4. **Show understanding** - Demonstrate knowledge of web development challenges
5. **Are respectful** - Professional language and constructive tone

**Common off-topic submissions (reject with "off-topic" label):**

- Browser troubleshooting or bug reports for specific sites
- Requests for help with personal projects
- General software or non-web technology requests
- Infrastructure or hosting questions
- Questions about using existing APIs (rather than improving them)

**If OFF-TOPIC:**

1. Add label: `off-topic`
2. Close issue
3. Comment: "Thank you for your submission. This request appears to be outside the scope of Web We Want, which focuses on improvements to web platform standards (HTML, CSS, JavaScript) and browser implementations. For support questions, please refer to appropriate community forums."

### Step 3: Technology Classification 🏷️

**Apply appropriate labels (select 1-3 most relevant):**

**Core Technologies:**

- `html` - HTML elements, attributes, semantic markup
- `css` - CSS properties, selectors, styling, layout
- `javascript` - JS language features, Web APIs, ECMAScript

**Specialized Areas:**

- `accessibility` - Screen readers, WCAG, inclusive design
- `dom` - Document Object Model, including window, navigator, CSSOM, etc.
- `api` - Web APIs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WebWeWant/webwewant.fyi](https://github.com/WebWeWant/webwewant.fyi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
