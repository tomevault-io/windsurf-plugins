---
trigger: always_on
description: These instructions define a unified style and process standard for authoring and maintaining learn.microsoft.com documentation with GitHub Copilot or other AI assistance.
---

# Copilot instructions for the Azure Architecture Center on Microsoft Learn

These instructions define a unified style and process standard for authoring and maintaining learn.microsoft.com documentation with GitHub Copilot or other AI assistance.

## Learn-wide Instructions

Below are instructions that apply to all Microsoft Learn documentation authored with AI assistance.

### AI usage & disclosure

All Markdown content created or substantially modified with AI assistance must include an `ai-usage` front matter entry:
- `ai-usage: ai-generated` – AI produced the initial draft with minimal human authorship
- `ai-usage: ai-assisted` – Human-directed, reviewed, and edited with AI support
- Omit only for purely human-authored legacy content

If missing, **add it**. However, do not add or update the ai-usage tag if the changes proposed are confined solely to:
- Links (link text and/or URLs)
- Single words or short phrases, such as entries in table cells
- Less than 5% of the article's word count

### Writing style

Follow [Microsoft Writing Style Guide](https://learn.microsoft.com/style-guide/welcome/) with these specifics:

#### Voice and Tone

- Active voice, second person addressing reader directly
- Conversational tone with contractions
- Present tense for instructions/descriptions
- Imperative mood for instructions ("Call the method" not "You should call the method")
- Use "might" instead of "may" for possibility
- Avoid "we"/"our" referring to documentation authors

#### Structure and Format

- Sentence case headings (no gerunds in titles)
- Be concise, break up long sentences
- Oxford comma in lists
- Number all ordered list items as "1." (not sequential numbering like "1.", "2.", "3.", etc.)
- Complete sentences with proper punctuation in all list items
- Avoid "etc." or "and so on" - provide complete lists or use "for example"
- No consecutive headings without content between them

#### Formatting Conventions

- **Bold** for UI elements
- `Code style` for file names, folders, custom types, non-localizable text
- Raw URLs in angle brackets

## About this repository

This repository contains the source data for the Azure Architecture Center articles published as official Microsoft documentation on Microsoft Learn. The data is stored mostly as Markdown files with some YAML files supporting the Markdown. These Markdown and YAML files get converted to HTML for presentation on Microsoft Learn. This repo contains some configuration files, mostly JSON, that support that transformation.

### Audience and how they use this data

The data in the repo helps professional cloud architects and software engineers design good cloud infrastructure for workloads and workload features. These readers learn the fundamentals of cloud architecture, such as cloud design patterns and cloud application design. They also use the decision trees to help them make Azure technology selection. Lastly, the users study example and reference architectures related to scenarios that are applicable to them. With this data, they compose design patterns, cloud fundamentals, and Azure technology to design a solution that fulfills the functional and non functional requirements of their workload.

### Repository facts

- This data gets published at <https://learn.microsoft.com/azure/architecture>.
- This is not a repository for software development.
- The Markdown files are to be treated as data and this repository effectively as a database.
- The data in this repository is grounded in Microsoft Azure technology.
- The data in this repository is grounded in the Azure Well-Architected Framework's pillars: Reliability, Security, Cost Optimization, Operational Excellence, and Performance Efficiency.
- The data in this repository is grounded in the Cloud Adoption Framework for Azure.
- This data must always lead to success of the person reading this.
- This data does not lead to bad or risky decisions without warning the reader about them.
- This data helps an architect avoid regret in their solution design.
- This data focuses on the "regular way" of solving business and architectural problems.
- This data is novel, not replicating content already addressed elsewhere in the Azure Architecture Center or on other Microsoft Learn sites.
- This data is truthful, even while being opinionated.
- This data does not showcase deprecated technology or solution approaches. It focuses on what durable greenfield success looks like. Deprecation notices are never a valid workaround.

### Repository structure

- This is the **private repository** (`-pr` suffix) for internal Microsoft authoring.
- A corresponding **public repository** exists at <https://github.com/MicrosoftDocs/architecture-center>.
- The `main` branch is for development; the `live` branch reflects published content.

### Your behavior

- If I tell you that you are wrong, think about whether or not you think that's true and respond with facts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MicrosoftDocs/architecture-center](https://github.com/MicrosoftDocs/architecture-center) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
