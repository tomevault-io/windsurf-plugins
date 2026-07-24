---
trigger: always_on
description: This document contains general and repository-specific instructions for GitHub Copilot when assisting with the `dotnet/AspNetCore.Docs` repository. **Unless otherwise specified, all ".NET" references refer to modern .NET, not .NET Framework.**
---


# Copilot Instructions for `dotnet/AspNetCore.Docs`

## Introduction
This document contains general and repository-specific instructions for GitHub Copilot when assisting with the `dotnet/AspNetCore.Docs` repository. **Unless otherwise specified, all ".NET" references refer to modern .NET, not .NET Framework.**

For code-specific guidelines, including code snippets, version targeting, and language standards, please refer to the [copilot-code-instructions.md](./copilot-code-instructions.md) file.

## General Guidelines

### 1. Issue Handling
When creating a PR for an issue:
* [ ] Read the full issue and all linked references
* [ ] Study code samples from linked PRs that demonstrate features in the latest .NET pre-release version (e.g., .NET 10 Preview) to ensure that guidance and documentation reflect the latest upcoming changes and best practices.
* [ ] For labeled issues that have the following labels, follow these guidelines:
  * [ ] **new-feature:** State which version introduced the feature
  * [ ] **bug:** Focus on correcting technical inaccuracies
* [ ] When you're assigned an issue, after you've completed your work and the workflows (status checks) have run, ensure there are no build warnings under the OpenPublishing.Build status check. If there are, open the build report (under View Details) and resolve any build warnings you introduced.
* [ ] Provide an overview of the project you're working on, including its purpose, goals, and any relevant background information.
* [ ] Include the folder structure of the repository, including any important directories or files that are relevant to the project.

### 2. Issue Discussion Analysis
When working on an issue:
* [ ] **Read the complete issue discussion** - Don't rely solely on the initial issue description
* [ ] **Prioritize maintainer guidance** - Comments from repository maintainers (especially those with "MEMBER" association) should take precedence over the original issue description
* [ ] **Look for updated analysis** - Later comments may contain revised understanding, additional context, or modified resolution approaches
* [ ] **Check for explicit instructions** - Look for phrases like "Action required by GitHub Copilot" or direct "@copilot" mentions that provide specific guidance
* [ ] **Validate your understanding** - If the discussion seems to contradict the initial issue description, follow the most recent maintainer guidance

### 3. Markdown File Naming and Organization
* [ ] If you're adding a new Markdown file, it should be named in all lowercase with hyphens separating words. Also, omit any filler words such as "the" or "a" from the file name.

### 4. API References and Verification
  * [ ] Use `<xref:api-doc-ID>` for API cross-references. 
  * [ ] The API documentation ID must be verified and sourced from the official XML documentation in dotnet-api-docs, never just infer API documentation IDs by looking for similar patterns.
  * [ ] If you cannot verify, state that explicitly in your output.

### 5. Links and References
* [ ] For cross-references to other articles within the AspNetCore.Docs repository:
  * [ ] Use the xref syntax: `<xref:target-uid>`
  * [ ] The "target-uid" of the xref syntax is obtained from the `uid` property value in the YAML front matter of the article's markdown file
  * [ ] Examples
    * [ ] For a target article `uid` value of `aspnetcore/mvc/overview`, the xref cross-link is `<xref:aspnetcore/mvc/overview>`
    * [ ] For a target article `uid` value of `blazor/index`, the xref cross-link is `<xref:blazor/index>`

* [ ] For non-markdown files (files that don't have the `.md` file extension) within this repository, such as PowerShell scripts and code files:
  * [ ] Use relative links with the appropriate file extension
  * [ ] Example: `../build-tools/build.ps1` or `./sample.json`

* [ ] For external links to non-Microsoft sites (MDN, W3C, etc.):
  * [ ] Use absolute URLs
  * [ ] Remove any language or culture segment from the URL path (such as `/en-us/`, `/fr-fr/`, `/en/`, etc.)
  * [ ] Example (MDN):  
    * [ ] Original: `https://developer.mozilla.org/en-US/docs/Web/API/Element/click_event`
    * [ ] Correct: `https://developer.mozilla.org/docs/Web/API/Element/click_event`

* [ ] For links to GitHub repositories:
  * [ ] Use the full URL path
    * [ ] Example: `https://github.com/maraf/blazor-wasm-react/blob/main/blazor/Counter.razor`
    * [ ] Example: `https://github.com/dotnet/blazor-samples/blob/main/10.0/BlazorWebAssemblyReact/blazor/Counter.razor`
  * [ ] For other Git hosting services or non-Microsoft domains, use the full URL
    * [ ] Example: `https://gitlab.com/username/repo-name`

* [ ] For links to Microsoft Learn content in other repositories:
  * [ ] Use the relative URL starting with a forward slash
  * [ ] Don't include the scheme and the host (example: `https://learn.microsoft.com`) and don't include the locale (example: `en-us`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnet/AspNetCore.Docs](https://github.com/dotnet/AspNetCore.Docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
