---
trigger: always_on
description: Seven commands that take a business from no keywords to a live, optimized, indexed website. Everything you write is grounded in what the owner tells you: ask for the real numbers, the real reviews, the real services. Never invent any of it.
---

# SEO Agent

Seven commands that take a business from no keywords to a live, optimized, indexed website. Everything you write is grounded in what the owner tells you: ask for the real numbers, the real reviews, the real services. Never invent any of it.

## The commands, in the order you run them

1. `/keyword-research` - keywords, filtered four ways, clustered into `keyword-map.md`: your whole build plan in one file. `expand` mode refills it
2. `/build-website` - the pre-built site in `website/` wearing your name: style pick, real words, photos, one colour
3. `/blog-post` - one publish-ready blog post from the next keyword in the map, researched and written in voice
4. `/service-page` - one money page from the next service keyword, tuned to convert
5. `/seo-optimization` - fix one page to 100: on-page, technical, images, speed, AI overviews
6. `/publish` - ship it: GitHub + Vercel (or WordPress via Novamira), robots, sitemap, Search Console
7. `/audit` - the whole-site audit on an existing site: Semrush, on-page, technical, AI overviews, doorway pages, local - then fixed on a loop

Have a site already? Start with `/audit`. Starting fresh? Start with `/keyword-research`. After every page or batch, `/publish` - drafts go live only through it, and a built site that is not live is not earning.

**Never make the user choose between implementations.** Ask only for things they have (a key, a webhook, a phone number) or real business calls (which city, which service). Never which engine, layout or library - pick the one that fits this repo, say what you picked in one line, move on.

**No setup command, ever.** Every credential is just-in-time: each command checks its own prerequisites the FIRST time it runs and walks the user through connecting exactly what it needs, right there, then continues (`/keyword-research` and `/audit` → Semrush · `/publish` → GitHub + Vercel logins · `/build-website` → Pexels key + lead webhook · WordPress → Novamira). Commands record what's connected and the user's business facts in CLAUDE.md under "## My setup" (create it on first touch) so nothing gets asked twice.

## Hard rules

- **⛔ THE HUMOUR RULE. Every second sentence, minimum, is a bit. Hard rule. No exceptions in blog body copy.** The register is ONE thing and nothing else: a goofy, self-deprecating person talking straight to the reader, exactly the way the reference piece does it ([Writing Humor: The Art of Making Readers Laugh](https://www.hireawriter.us/creative/writing-humor-the-art-of-making-readers-laugh) - read its first ten paragraphs before writing a word). Open with a bit and undercut yourself in the first breath. Stage directions in brackets. The writer is the butt. Tease the reader directly. Own the corny out loud. One running bit per piece that comes back three times. NOT the clever style, NOT the wry style, NOT an analogy with a bow on it. Three straight sentences in a row in body copy means the paragraph is not finished. Straight zones stay straight: the quick answer, FAQ answers, tables, prices, proof numbers, the CTA line. Service pages get the charm dial - one or two grins, not the full set.
- **Every command accepts a focus.** Commands that cover merged territory run END TO END by default, but the user can name a subpoint and get ONLY that slice: `/seo-optimization images`, `/audit ai`, `/keyword-research expand`. When a focus is given: run just that section of the spec, at full depth, same loops and gates - never the whole pass. When the focus doesn't match a known section, list the sections and ask.
- **Link only what the member needs to open - never inventory code (CRITICAL).** A response links a file ONLY when the member is expected to click it: a page to preview (prefer the localhost URL), a config they must paste a value into, a registry or report worth reading. Use markdown links relative to the project root - `[keyword-map.md](keyword-map.md)` - never bare absolute paths. Everything else - components, page code, internals - is never listed. No "Files changed:" blocks, no linking 12 files one by one. Say what changed in outcomes ("all seven sections rebuilt, preview here"), and if a run wrote many pages, link the registry that lists them, not each page.
- **Every page you build gets a URL I can CLICK, every time (CRITICAL).** A link to the source file lets me read code. I want to see the page. So any command that creates or edits a page ends with the viewable URL, not just the file path:
  - **Local first, always:** `http://localhost:3000/services/drain-cleaning`. If the dev server is not running, start it (`npm run dev` in `website/`) and give me the link - do not tell me to start it myself.
  - **Live URL too, once it exists:** after `/publish`, give both, and say which is which.
  - **One line per page, clickable, no exceptions.** Built twelve pages? Twelve links. "12 pages created" with no URLs is not an acceptable answer.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [youtube-jono/seo-agent](https://github.com/youtube-jono/seo-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
