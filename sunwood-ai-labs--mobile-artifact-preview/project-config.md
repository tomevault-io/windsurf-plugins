---
trigger: always_on
description: Use when exposing local development artifacts to a phone or tablet, previewing Codex or agent outputs from mobile, troubleshooting LAN-accessible viewers, Nextcloud previews, file links, screenshots, generated images, HTML, SVG, JSON, XML, Markdown, PDF, or other artifacts that the user needs to visually confirm from a mobile device.
---


# Mobile Artifact Preview

## Purpose

Use this skill to make local development artifacts visible from mobile devices and to prove that they render. The deliverable is not only a local path: provide a clickable mobile-accessible link, verify the real display surface when possible, and include the exact local fallback path.

This skill supports workflows such as:

- Showing generated images, screenshots, diagrams, HTML, SVG, Markdown, JSON, XML, CSV, PDF, or draw.io exports to the user.
- Creating or repairing a LAN file viewer, Nextcloud mount, or preview plugin for local project files.
- Verifying mobile layout for generated previews or local web apps.
- Storing evidence screenshots under a project folder so the user can inspect them later.

## Core Rule

When the user asks to show, preview, display, or confirm an artifact, treat the visible mobile-checkable surface as the deliverable.

For every newly created or updated user-checkable artifact, especially generated
images, screenshots, HTML, SVG, JSON/XML previews, PDFs, diagrams, reports, or
other visual outputs, include a clickable URL in the chat whenever a LAN viewer
or Nextcloud route is available. Do not make the user ask separately for the
link.

Do not stop at "file created" or a raw local path. Return:

1. A clickable link if a LAN viewer or Nextcloud route is available.
2. The local filesystem path as fallback.
3. The verification surface used, such as `view_image`, Browser screenshot, Playwright mobile viewport, WebDAV listing, or Nextcloud app check.

For images shown in chat, also call `view_image` on the exact file before saying it was displayed.

When a generated image, repository header, thumbnail, or release banner needs
visible text and the user asked for image generation, do not add that text later
with Pillow, SVG, canvas, or another post-processing overlay unless the user
explicitly approves that production method. Treat generated typography as part
of the deliverable: inspect the image-gen output, retry if the text is wrong,
and record whether the final asset is direct image-gen output or a post-processed
composite.

## Preferred Surfaces

Choose the narrowest surface that fits the artifact:

| Need | Preferred surface |
| --- | --- |
| Image visible in chat | `view_image` on the exact file |
| Mobile-accessible project file | Nextcloud file viewer |
| Mobile web UI or preview app | LAN URL plus mobile viewport screenshot |
| MP4/video visible on phone | Verified video viewer or direct HTTP video endpoint with mobile viewport screenshot |
| Local-only static HTML | Start or reuse a small LAN web server |
| JSON/XML structure | Structured preview or companion `.html` preview |
| Repeated screenshots/evidence | Project-local `evidence/` folder |

## Nextcloud Viewer Package

When the bundled Nextcloud viewer is available, prefer it for project and Codex
artifact links:

```text
Default local URL: http://127.0.0.1:8793/
Phone-facing URL: ${MOBILE_ARTIFACT_NEXTCLOUD_BASE_URL:-https://<tailscale-host>:8443}
Default login: admin / admin
Project mount: ${MOBILE_ARTIFACT_PROJECTS_DIR:-${HOME}/Prj} -> /Project
Codex mount: ${MOBILE_ARTIFACT_CODEX_DIR:-${HOME}/.codex} -> /Codex
Project folder link pattern:
${MOBILE_ARTIFACT_NEXTCLOUD_BASE_URL}/apps/files/files?dir=/Project/<path-under-projects-dir>
```

For links returned to the user, prefer the configured HTTPS base URL. Do not
paste old LAN `http://192.168...:8793` links when HTTPS/Tailscale Serve is
configured. On this Mac, the verified Nextcloud phone-facing base URL is:

```text
https://macbook-pro-von-admin.tail8be30.ts.net:8443
```

The bare `https://macbook-pro-von-admin.tail8be30.ts.net` route may point to a
different preview service, so keep the `:8443` port for Nextcloud links unless
`tailscale serve status` proves the mapping changed.

Bundled implementation source in this repository:

```text
assets/nextcloud-file-viewer
```

Useful checks:

```bash
cd assets/nextcloud-file-viewer
docker compose ps
docker exec -u www-data agent-nextcloud php occ app:list | rg 'structuredviewer|htmlviewer|text|viewer|pdf'
docker exec -u www-data agent-nextcloud php occ files:scan --path='admin/files/Project/<relative-folder>' --shallow
```

For a folder link under the projects directory, return:

```text
${MOBILE_ARTIFACT_NEXTCLOUD_BASE_URL}/apps/files/files?dir=/Project/<relative-folder>
```

## Workflow

1. Identify the artifact path and file type.
2. If the artifact is an image intended for chat, call `view_image` on that exact path.
3. If the artifact is under the configured projects directory, build the Nextcloud `/Project/...` link and shallow-scan the folder if new files may not be indexed.
4. If the artifact is under the configured Codex directory, build the Nextcloud `/Codex/...` link when the mount is available.
5. Include that clickable URL in the same chat response as the generated artifact, before or next to the local path.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sunwood-ai-labs/mobile-artifact-preview](https://github.com/Sunwood-ai-labs/mobile-artifact-preview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
