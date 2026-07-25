---
trigger: always_on
description: This document contains general and repository-specific instructions for GitHub Copilot when assisting with the `cpp-docs-pr` repository.
---

# Copilot Instructions for `cpp-docs-pr`

## Introduction

This document contains general and repository-specific instructions for GitHub Copilot when assisting with the `cpp-docs-pr` repository.

## Priority Levels

This document uses the following markers to indicate instruction priority:

- **[REQUIRED]** - Must be followed. May cause build failures or PR blocks if violated
- **[STYLE]** - Microsoft Writing Style Guide requirement for consistency
- **[C++-SPECIFIC]** - C++ documentation convention that overrides general guidance
- No marker - Best practice recommendation; use judgment based on context

## General Guidelines

### Using MCP tools
If the user intent relates to Azure Devops, make sure to prioritize the Azure DevOps MCP server tools.

### 1. Issue Handling
When creating a PR for an issue:
- [ ] Read the full issue and all linked references
  - [ ] **new-feature:** State which version introduced the feature
  - [ ] **bug:** Focus on correcting technical inaccuracies
- [ ] When you're assigned an issue, after you've completed your work and the workflows (status checks) have run, ensure there are no build warnings under the OpenPublishing.Build status check. If there are, open the build report (under View Details) and resolve any build warnings you introduced.
- [ ] When starting work on an issue, document your understanding in a comment: state the issue's purpose, expected outcome, and your implementation approach.

### 2. Issue Discussion Analysis
When working on an issue:
- [ ] **Read the complete issue discussion** - Don't rely solely on the initial issue description
- [ ] **Prioritize maintainer guidance** - Comments from repository maintainers (especially those with "MEMBER" association) should take precedence over the original issue description
- [ ] **Look for updated analysis** - Later comments may contain revised understanding, additional context, or modified resolution approaches
- [ ] **Check for explicit instructions** - Look for phrases like "Action required by GitHub Copilot" or direct "@copilot" mentions that provide specific guidance
- [ ] **Validate your understanding** - If the discussion seems to contradict the initial issue description, follow the most recent maintainer guidance

### 3. File Naming and Organization
**Naming conventions:**
- [ ] Name new Markdown files in all lowercase with hyphens separating words
- [ ] Omit filler words like "the" or "a" from file names

**Folder structure:**
- [ ] Linux topics → `docs/linux/`
- [ ] C++ STL → `docs/standard-library/`
- [ ] C runtime → `docs/c-runtime-library/`
- [ ] C++ language → `docs/cpp/`
- [ ] Visual Studio IDE features → `docs/ide/`
- [ ] Build process/modules → `docs/build/reference/`
- [ ] Build Insights → `docs/build-insights/`

### 4. Links and References
- [ ] Add links to related topics and resources where appropriate.
- [ ] Use relative links for files in this repo 
- [ ] **[REQUIRED]** Links to other documentation articles should be relative, not absolute. Start relative links with `/docs/` and include the `.md` suffix. If you add a link to another page on learn.microsoft.com that's not in this repo, remove https://learn.microsoft.com/en-us from the link.
- [ ] **[REQUIRED]** Links to bookmarks within the same article should be relative and start with `#`.
- [ ] **[REQUIRED]** Link descriptions should be descriptive and make sense on their own. Don't use "click here" or "this link" or "here".
- [ ] **[STYLE]** When you are going to refer to another file or an article on the web, use this format: "For more information, see [descriptive name of link](link path)." The exception to this is the See Also links at the end of an article. Those should be markdown links and contain the title of the article you link to as the descriptive portion of the link.
- [ ] For external links to non-Microsoft sites:
  - [ ] **[REQUIRED]** Use absolute URLs
  - [ ] **[REQUIRED]** Remove any language or culture segment from the URL path (such as `/en-us/`, `/fr-fr/`, `/en/`, etc.)
  - [ ] Example (MDN):  
    - [ ] Original: `https://developer.mozilla.org/en-US/docs/Web/API/Element/click_event`
    - [ ] Correct: `https://developer.mozilla.org/docs/Web/API/Element/click_event`

- [ ] For links to Microsoft Learn content in other repositories:
  - [ ] **[REQUIRED]** Use the relative URL starting with a forward slash
  - [ ] Don't include the scheme and the host (example: `https://learn.microsoft.com`) and don't include the locale (example: `en-us`)
  - [ ] Example: For the target Learn website URL `https://learn.microsoft.com/en-us/dotnet/core/introduction`, use the relative URL `/dotnet/core/introduction` for the link destination

## Repository-Specific Guidelines
- [ ] **[REQUIRED]** Follow the [Microsoft Writing Style Guide](https://learn.microsoft.com/en-us/style-guide/welcome/)
  - [ ] Use contractions following the guidance in [Use contractions](https://learn.microsoft.com/en-us/style-guide/word-choice/use-contractions)
- [ ] **Repository Exceptions**:
  - [ ] **[REQUIRED]** Use **bold** text for UI elements like menu items, dialog names, and names of text boxes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MicrosoftDocs/cpp-docs](https://github.com/MicrosoftDocs/cpp-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
