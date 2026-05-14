---
trigger: always_on
description: MPA-First Development Mandate - Strict rules for building performant, resilient Multi-Page Applications
---


# AI Coding Agent Guidelines: The MPA-First Mandate

## Introduction: Why We Build This Way

For over a decade, web development has trended towards complex, JavaScript-heavy Single-Page Applications (SPAs). The promise was a slick, "app-like" user experience. The reality, in most cases, has been bloated, fragile, and over-engineered websites that are slow to load, difficult to maintain, and hostile to users and search engines.

This document outlines a return to a more resilient, performant, and durable web. We are intentionally choosing a **Multi-Page Application (MPA)** architecture. The guiding principle is to **use the platform**. We build upon the native strengths of HTML, CSS, and the browser itself, which have evolved significantly. Modern features like CSS View Transitions and Speculation Rules now provide the fluid user experience that once required megabytes of JavaScript, but without the performance penalty.

We are choosing simplicity, speed, and maintainability over unnecessary complexity. We build for users and outcomes, not for developer experience (DX) or architectural novelty. JavaScript is a powerful tool for progressive enhancement, not the default foundation for every page.

---

## PRIME DIRECTIVE: MPA-Only Architecture

- You **MUST** create server-rendered, multi-page websites (MPAs). Each distinct page or view must be its own unique file (e.g., `.html`, `.php`) served at a distinct URL.
- Navigation **MUST** use standard anchor links (`<a href="...">`) that trigger a full page navigation. The browser will handle the request, and the server will respond with a new HTML document.
- The goal is to produce sites that are fundamentally functional with zero client-side JavaScript. JS should only be used to enhance, not to enable, core functionality.

### 🚫 BANNED TECHNOLOGIES & PATTERNS

- **DO NOT** use or reference any Single-Page Application (SPA) frameworks, libraries, or patterns.
  - This includes, but is not limited to: React, Angular, Vue, Svelte, Next.js, Nuxt.js.
- **DO NOT** use JSX, TSX, TypeScript-based routing, or any form of client-side routing.

---

## Guiding Complex Changes

When approaching a complex change or refactoring a large file, prioritize clarity and communication:

1. **Outline a Plan:** Before diving in, briefly describe your approach. What is the goal? Which parts of the code will you touch?
2. **Communicate as You Go:** Explain your changes in small, logical steps. This allows for feedback and course correction without rigid, multi-step approval gates.
3. **Focus on Conceptual Changes:** Group your edits logically. For example, a commit might be "Refactor user authentication logic," not "Change 15 different files."

---

## Folder Structure

This structure promotes a clean separation of concerns and follows the Model-View-Controller (MVC) architectural pattern:

```
project-root/
├── public/                # Web root, all publicly accessible files
│   ├── assets/
│   │   ├── css/
│   │   ├── js/
│   │   ├── images/
│   │   ├── fonts/
│   └── index.php          # Or index.html
├── src/                   # Application source code
│   ├── controllers/       # Handles user requests
│   ├── models/            # Business logic and data interaction
│   ├── views/             # HTML templates/partials
│   └── utilities/         # Helper functions, etc.
├── vendor/                # Composer dependencies
├── config/                # Configuration files
├── tests/                 # Automated tests
└── docs/                  # Project documentation
```

---

## SEO Best Practices: A Top Priority

Excellent SEO is not an afterthought; it's a direct result of building a clean, semantic, and performant MPA.

### 1. The Head is Everything

Ensure every page has a comprehensive and valid `<head>` section:

**Example: Detailed `<head>` for a Blog Post**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>It's Time for Modern CSS to Kill the SPA | My Awesome Blog</title>
    <meta name="description" content="Native CSS transitions have quietly killed the strongest argument for client-side routing. Learn how to build faster, simpler websites.">

    <link rel="canonical" href="https://www.example.com/blog/css-kills-spa">

    <meta property="og:title" content="It's Time for Modern CSS to Kill the SPA">
    <meta property="og:description" content="Native CSS transitions have quietly killed the strongest argument for client-side routing. Learn how to build faster, simpler websites.">
    <meta property="og:type" content="article">
    <meta property="og:url" content="https://www.example.com/blog/css-kills-spa">
    <meta property="og:image" content="https://www.example.com/assets/images/blog/og-image-css-spa.jpg">
    <meta property="og:image:width" content="1200">
    <meta property="og:image:height" content="630">
    <meta property="og:site_name" content="My Awesome Blog">

    <meta name="twitter:card" content="summary_large_image">
    <meta name="twitter:site" content="@MyAwesomeBlog">
    <meta name="twitter:title" content="It's Time for Modern CSS to Kill the SPA">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RealistSec/mpa-first-guidelines](https://github.com/RealistSec/mpa-first-guidelines) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
