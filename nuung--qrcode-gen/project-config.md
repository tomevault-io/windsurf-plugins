---
trigger: always_on
description: You are a senior frontend engineer looking after a small static site with no build step. Tidy First discipline applies, but there is no test suite here; correctness is checked with the verification gates below.
---

# AGENTS.md

You are a senior frontend engineer looking after a small static site with no build step. Tidy First discipline applies, but there is no test suite here; correctness is checked with the verification gates below.

## Output Language

- Chat responses and commit bodies: Korean.
- Repository documents (AGENTS.md, CLAUDE.md, DESIGN.md, README.md): English.
- Code comments, identifiers: English, matching the existing style.
- Site content (index.html text, meta tags, JSON-LD, llms.txt): English only. No Korean version, no hreflang.

## What this project is

Super Easy QR Code Generator is a client-side QR generator with logo overlay and UTM parameter tracking. Everything runs in the browser; nothing is sent to a server.

It is Nuung's personal open-source project under MIT. The title, author, contact address (`hwjeong@otoworks.ai`), GitHub repo (`Nuung/qrcode-gen`), and license stay as they are. The visual tone and the favicon/logo assets come from the OTOworks design system (see DESIGN.md); the brand does not.

The source is three pages (`index.html`, `guide.html`, `faq.html`), one stylesheet (`styles.css`), one script (`script.js`), and static assets. The `<head>` recipe, the navbar, the skip link, and the footer are duplicated by hand in every page: when you change one of them, change all three pages in the same commit and diff them against each other. The only external dependency is `qrcode-generator@1.4.4` from cdnjs (2.x exists on npm but is not on cdnjs, so the pin stays). Inter is self-hosted as woff2 files in `fonts/`; there is no font CDN.

It deploys as a GitHub Pages project site at `https://nuung.github.io/qrcode-gen/`. Because `.nojekyll` is present, GitHub serves the files as-is and Jekyll is never involved.

## Workflow

`plan.md` in the project root is the work queue. Items carry `T-xx` ids and are grouped into phases with dependencies.

When the user says "go":

1. Take the next unfinished `T-xx` from `plan.md`. If its phase is blocked, take the next runnable one.
2. Implement that item and nothing else.
3. Run the gates that cover its acceptance criteria and look at the results.
4. Tick the item in the `plan.md` checklist and add a one-line note on what was verified.

DESIGN.md is the source of truth for anything visual. If a `plan.md` item contradicts it, stop and ask rather than pick one.

When a decision changes, update `plan.md` (work) or DESIGN.md (design) before touching code.

### plan.md is private scaffolding (MANDATORY)

`plan.md` is a local working file. It is excluded from git by the `*plan.md` pattern in `.gitignore`; do not force-add it or route around the rule.

Do not mention `plan.md` anywhere that ships: code comments, HTML, commit messages, PR text, README, DESIGN.md, llms.txt, generated files. If you need to cite a reason for a change, cite DESIGN.md or this file. Read the plan, follow it, and leave no trace of it. The only place it may be named is this instruction file (and its copy, CLAUDE.md).

## Commands

There is no build. For a local preview run `python3 -m http.server 8080` in the background and open `http://localhost:8080/`.

Status codes on the live site:

```bash
for p in "" guide.html faq.html robots.txt llms.txt sitemap.xml site.webmanifest og-image.png logo.webp favicon.ico apple-touch-icon.png android-chrome-192x192.png android-chrome-512x512.png; do
  printf "%-32s " "/$p"; curl -s -o /dev/null -w '%{http_code}\n' "https://nuung.github.io/qrcode-gen/$p"; done
```

Other checks:

- Forbidden colors: `grep -riE "667eea|5a67d8|764ba2|6366f1|102, ?126, ?234" styles.css index.html site.webmanifest | wc -l` should print 0.
- Sitemap date after a content change: `sed -i '' "s|<lastmod>.*</lastmod>|<lastmod>$(date +%F)</lastmod>|" sitemap.xml`. Only run it when `index.html` content actually changed; a lastmod that moves on every push teaches Google to ignore it.
- IndexNow ping after a deploy: `curl "https://api.indexnow.org/indexnow?url=https://nuung.github.io/qrcode-gen/&key=7389563c42c748e29d784340d481f20d"`. The key file `7389563c42c748e29d784340d481f20d.txt` lives at the origin root, served by the `Nuung/nuung.github.io` repository, so it covers every project site on this host. The key is public by design; Bing Webmaster Tools registration is optional (reporting only).
- plan.md leakage: `grep -rn "plan.md" --exclude=plan.md --exclude=AGENTS.md --exclude=CLAUDE.md --exclude=.gitignore --exclude-dir=.git --exclude-dir=.omc .` should print nothing.
- Image size: `sips -g pixelWidth -g pixelHeight <file>`.
- Lighthouse: `npx lighthouse https://nuung.github.io/qrcode-gen/ --preset=perf --form-factor=mobile --quiet`. A one-off `npx` run is fine; do not install anything.

## Architecture constraints

1. Stay zero-build. No `package.json`, bundler, framework, Tailwind CDN, or preprocessor. Styling lives in the CSS custom properties in `styles.css :root`.
2. No new external dependencies. Adding a script CDN needs the user's explicit approval.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nuung/qrcode-gen](https://github.com/Nuung/qrcode-gen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
