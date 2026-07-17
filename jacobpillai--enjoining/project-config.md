---
trigger: always_on
description: A client-side adult video streaming and library management platform built with Vite + Vanilla JavaScript ES Modules, HTML5, CSS3, and Bootstrap 5. No traditional backend — all data is driven by JSON metadata files and managed client-side. The platform is intended as a **controlled, private-access environment** and is not publicly available.
---

# GitHub Copilot Instructions — JOI Game / Adult Video Platform

## Project Overview

A client-side adult video streaming and library management platform built with Vite + Vanilla JavaScript ES Modules, HTML5, CSS3, and Bootstrap 5. No traditional backend — all data is driven by JSON metadata files and managed client-side. The platform is intended as a **controlled, private-access environment** and is not publicly available.

---

## Tech Stack

| Layer       | Technology                                              |
| ----------- | ------------------------------------------------------- |
| Build Tool  | Vite (ES Modules, HMR)                                  |
| Language    | Vanilla JavaScript (ESM, no TypeScript)                 |
| UI          | HTML5, CSS3, Bootstrap 5                                |
| Media       | HTML5 `<video>`, Web Audio API                        |
| Data        | JSON files (metadata, user config, library)             |
| Permissions | Client-side role system via sessionStorage/localStorage |
| Downloads   | yt-dlp (external CLI), native `<a download>`links     |

---

## Project Structure Conventions

```
/public
  /videos          → Local video files
  /assets          → Thumbnails, posters, icons
  video-metadata.json → Master video library manifest

/src
  /modules         → ES module files (one responsibility each)
  /components      → Reusable UI builders (pure functions returning DOM elements)
  /styles          → CSS per-component or per-page
  /data            → JSON loaders and schema validators
  /auth            → Role/permission logic
  main.js          → Vite entry point
  router.js        → Client-side routing (hash or History API)

/urls.txt          → Plaintext list of external URLs for yt-dlp batch downloads
```

---

## Coding Style & Patterns

* **Always use ES Modules** — `import/export`, never `require()` or `<script>` globals.
* **No frameworks** — do NOT suggest React, Vue, or Angular. Vanilla JS only.
* **No TypeScript** — plain `.js` files only.
* **Bootstrap for layout** — use Bootstrap 5 grid, utility classes, modals, and cards. Do NOT reinvent layout from scratch.
* **Functional style preferred** — prefer pure functions and module-level state over classes where practical.
* **DOM manipulation** — use `document.createElement`, `insertAdjacentHTML`, or template literals. Avoid `innerHTML` with unsanitized user input.
* **Async/await** — always use `async/await` over `.then()` chains for fetch and file reads.
* **Event delegation** — attach events to parent containers where lists are dynamic.

---

## JSON Metadata Schema

All videos are described by entries in `video-metadata.json`. Always follow this schema:

```json
{
  "id": "uuid-v4-string",
  "title": "Video Title",
  "description": "Optional description",
  "filename": "video-file.mp4",
  "url_external": "https://...",
  "thumbnail": "/assets/thumb.jpg",
  "duration": 1234,
  "tags": ["category1", "tag2"],
  "performers": ["Name One", "Name Two"],
  "platform": "PornHub | XVideos | Local | PMVHaven",
  "download_urls": {
    "1080p": "https://...",
    "720p":  "https://...",
    "480p":  "https://..."
  },
  "added_at": "ISO8601 timestamp",
  "visible_to": ["admin", "member"],
  "uploaded_by": "username"
}
```

* **Never skip `id`, `title`, `visible_to`, or `added_at`** — these are required.
* `visible_to` is an array of role names that can see this entry.
* When generating functions that read metadata, always validate these fields before use.

---

## User Roles & Permissions System

The platform uses a **client-side role system** stored in `sessionStorage`. There is no server-side auth — this is a controlled private environment.

### Roles (in ascending access level):

| Role       | Access                                                    |
| ---------- | --------------------------------------------------------- |
| `guest`  | View public-tagged content only, no downloads             |
| `member` | View member + public content, single-quality download     |
| `admin`  | Full library access, multi-quality downloads, JSON upload |

### Pattern to follow:

```js
// src/auth/permissions.js
export function getCurrentUser() {
  return JSON.parse(sessionStorage.getItem('currentUser')) ?? { role: 'guest' };
}

export function can(action, user = getCurrentUser()) {
  const rules = {
    guest:  ['view:public'],
    member: ['view:public', 'view:member', 'download:single'],
    admin:  ['view:public', 'view:member', 'view:admin',
             'download:single', 'download:multi', 'upload:json', 'manage:library'],
  };
  return rules[user.role]?.includes(action) ?? false;
}
```

* Always call `can('action')` before rendering gated UI elements.
* Never expose admin controls to non-admin roles in the DOM — conditionally render, don't just hide with CSS.

---

## Video Library & Categorization

* The library is loaded from `video-metadata.json` at startup and cached in a module-level variable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JacobPillai/Enjoining](https://github.com/JacobPillai/Enjoining) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
