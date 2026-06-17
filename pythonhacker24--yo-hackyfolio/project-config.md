---
trigger: always_on
description: This is a data-driven Next.js portfolio. All content lives in one file. Components are fixed; data is not.
---

# Hackyfolio — Agent Guide

This is a data-driven Next.js portfolio. All content lives in one file. Components are fixed; data is not.

## The one rule

**Edit content in `app/data/portfolio.json`. Do not hardcode content inside components.**

## How the system works

```
app/data/portfolio.json
  -> SectionRenderer (app/components/sections/registry.tsx)
    -> one component per section type
      -> rendered on the page
```

The page (`app/page.tsx`) loops over the `sections` array and renders each one. The agent-mode Markdown view is generated from the same JSON by `app/data/generateMarkdown.ts`.

## The JSON structure

```
portfolio.json
  meta        -> siteUrl, calendarUrl, email
  socials[]   -> label, href, icon (used in navbar + contact section)
  sections[]  -> ordered list of sections (order = render order on page)
```

Each section:
```json
{
  "type": "sectionType",
  "title": "Section Heading",
  "data": { ... }
}
```

The `hero` section has no `title` field. Everything else does.

## Section types and their data shapes

Read the exported `*Data` type at the top of each component file for the exact schema. Here is a quick reference:

### `hero`
File: `app/components/sections/Hero.tsx`
```json
{
  "type": "hero",
  "data": {
    "image": "/me.png",
    "name": "Your Name",
    "phonetic": "/fəˈnetɪk/",
    "noun": "noun",
    "timezone": { "label": "IST", "tz": "Asia/Kolkata" },
    "intro": ["paragraph one", "paragraph two"]
  }
}
```

### `experience`
File: `app/components/sections/ExperienceSection.tsx`
```json
{
  "type": "experience",
  "title": "Experience",
  "data": {
    "featured": {
      "name": "Company Name",
      "link": "https://...",
      "role": "Role, Location",
      "dateRange": "Month Year - Present",
      "collapsedHeight": "max-h-48",
      "body": [ ...blocks ]
    },
    "previousLabel": "Previously",
    "previous": [
      {
        "name": "Company Name",
        "role": "Role, Location",
        "link": "https://...",
        "body": [ ...blocks ]
      }
    ]
  }
}
```

### `techStack`
File: `app/components/sections/TechStackSection.tsx`
```json
{
  "type": "techStack",
  "title": "Section title",
  "data": {
    "categories": [
      {
        "name": "Languages",
        "skills": [
          { "name": "Go", "slug": "go" }
        ]
      }
    ]
  }
}
```
The `slug` must match a valid icon on [simpleicons.org](https://simpleicons.org). Check the site if unsure.

### `expandableCard`
File: `app/components/sections/ExpandableCardSection.tsx`
```json
{
  "type": "expandableCard",
  "title": "Section title",
  "data": {
    "heading": "Card heading",
    "collapsedHeight": "max-h-48",
    "body": [ ...blocks ]
  }
}
```

### `project`
File: `app/components/sections/ProjectSection.tsx`
```json
{
  "type": "project",
  "title": "Section title",
  "data": {
    "name": "Project Name",
    "link": "https://...",
    "subtitle": "short tagline",
    "body": [ ...blocks ],
    "stats": [
      { "value": "201", "label": "sign-ups in 3 days" }
    ],
    "footerLink": { "label": "Read more", "url": "https://..." }
  }
}
```
`stats`, `link`, `subtitle`, and `footerLink` are all optional.

### `youtube`
File: `app/components/sections/YouTubeSection.tsx`
```json
{
  "type": "youtube",
  "title": "My YouTube Channel",
  "data": {
    "image": "/youtube-profile.png",
    "name": "Channel Name",
    "url": "https://youtube.com/@handle",
    "tagline": "short description",
    "community": {
      "url": "https://discord.gg/...",
      "count": "300+ members",
      "text": "in the Discord"
    },
    "videos": [
      { "title": "Video title", "url": "https://youtube.com/watch?v=..." }
    ],
    "footerLink": { "label": "See all videos", "url": "https://..." }
  }
}
```

### `education`
File: `app/components/sections/EducationSection.tsx`
```json
{
  "type": "education",
  "title": "Education",
  "data": {
    "items": [
      {
        "title": "Institution Name",
        "role": "Degree / Field",
        "link": "https://...",
        "body": [ ...blocks ]
      }
    ]
  }
}
```

### `github`
File: `app/components/sections/GithubSection.tsx`
```json
{
  "type": "github",
  "title": "GitHub Contributions",
  "data": { "username": "YourGitHubUsername" }
}
```

### `publications`
File: `app/components/sections/PublicationsSection.tsx`
```json
{
  "type": "publications",
  "title": "Research Publications",
  "data": {
    "items": [
      {
        "title": "Paper title",
        "link": "https://doi.org/...",
        "venue": "Conference or journal name",
        "authors": "Name A; Name B",
        "abstract": "Full abstract text here.",
        "collapsedHeight": "max-h-32",
        "linkLabel": "View Publication"
      }
    ]
  }
}
```

### `recommendations`
File: `app/components/sections/RecommendationsSection.tsx`
```json
{
  "type": "recommendations",
  "title": "Recommendations",
  "data": {
    "items": [
      {
        "name": "Person Name",
        "link": "https://linkedin.com/in/...",
        "role": "Their title / context",
        "quote": ["paragraph one", "paragraph two"]
      }
    ]
  }
}
```

### `contact`
File: `app/components/sections/ContactSection.tsx`
```json
{
  "type": "contact",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PythonHacker24/yo-hackyfolio](https://github.com/PythonHacker24/yo-hackyfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
