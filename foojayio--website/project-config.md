---
trigger: always_on
description: generates has none — **`with .File` is the whole guard**. It is a URL builder
---

# Project context for Claude and other LLM coding tools

foojay.io is a static Hugo site, built from this repo and deployed to GitHub
Pages behind Cloudflare. It replaced the WordPress site at **cutover on
2026-09-22**. If you're picking this up fresh, read this before making changes.

`REDIRECTS.md` documents the live Cloudflare redirect rules that carry old
URLs to their Hugo equivalents.

## The goal that outranks the others

**Publishing a post has to stay effortless for the author.** Contributors send
posts as pull requests (see `CONTRIBUTING.md`); most of them write Java, not
Hugo, and they should be able to open a file, write Markdown, and be done.
Every flag, frontmatter key, naming rule or manual step is a tax on that, and a
thing an author can get wrong or forget.

So **validate every change against this**, and prefer, in order:

1. **Derive it.** If the build can work it out from the content, it must —
   don't ask the author. The layout detects code blocks in the rendered page
   instead of reading an `enlighterjs:` flag; sponsor article counts and
   "Topics covered" are computed from `authors:` rather than stored.
2. **Default it.** If it can't be derived, pick the right default and let the
   rare case override.
3. **Ask for it.** Only when the answer genuinely lives in the author's head
   (`title`, `related_posts`, a sponsor's `authors:` list).

A flag that is always set to the same value is not configuration, it's a
chore — delete it. When a knob does have to exist, `validate/Frontmatter.java`
should catch a mistake at PR time rather than letting it fail silently.

## Hard requirement: don't overload comments

A comment earns its place by saying what the code cannot. Keep new ones short:

- **Say WHY, not what** — the code already says what it does.
- **Three or four lines is the ceiling**, and one is usually enough. Only a
  genuinely subtle trap earns more.
- **Say it once.** If something is already explained elsewhere, point at it
  rather than repeating it.
- **No narrating the diff**, and no history of what the code used to do.

This applies to every language here: Go templates, CSS, JS, Java, frontmatter.
Plenty of existing comments are longer than this allows — shorten them when you
touch them, and don't take them as the model for new ones.

## What exists

- **Hugo skeleton**: `hugo.toml`, `themes/foojay/` (layouts +
  `static/css/style.css`), and `template/` (starter files for article, page,
  author, board member, ad and event, plus the category list — see
  `template/README.md`). There is deliberately **no `archetypes/`**: nothing
  runs `hugo new`, and two sets of starter files drifted — the post archetype
  wrote a singular `author:` against author *files* where posts take an
  `authors:` list of author *folders*. Add starter files to `template/`.

- **`scripts/` is grouped by lifetime, not by verb** — `fetch/` (external data,
  runs in CI) and `validate/` (PR-time checks). A script is named for **what it
  produces**, the folder supplying the verb (`fetch/Jugs.java`, not
  `FetchJugs.java`). All run from the repo root — they resolve `content/` and
  `data/` against the working directory. `scripts/README.md` is the index.

  `transfer/`, `cleanup/` and the `shared/` converter they both called were
  deleted at cutover. Where a convention below names `HtmlToMarkdown`,
  `Posts.java` or `cleanup/images.py`, that is one of them: they are named
  because they explain the shape of what is in `content/`, not because there is
  a file to open. Read one out of git history if you need it.

- **`content/` came out of a scraper, not a database export.** The deleted
  `transfer/` scripts read the live foojay.io site (no WP admin or DB access)
  and wrote the Markdown in the repo today. When something in `content/` looks
  odd, the answer is usually "this is what the WordPress page rendered".

### Data fetchers (`scripts/fetch/`)

All run daily from `sync-external-content.yml`, commit their result, then
**dispatch** a deploy (see workflows below).

- **`Jugs.java`** regenerates `data/jugs.yaml` from the community
  [World Wide JUGs directory](https://github.com/World-Wide-JUGs/GlobalWWJugs).
  JUG leads fix their own entry upstream. Derives `meetup_slug`/`meetup_url`
  when a JUG's `website` is a meetup.com URL.

- **`JavaChampions.java`** regenerates `data/java-champions.yaml` from
  [aalmiray/java-champions](https://github.com/aalmiray/java-champions), and
  resolves the coordinates behind the `/java-champions/` map from three sources
  in order: an upstream `location: {lat, lon}`; `data/geocode-cache.yaml`; then
  [geocode.maps.co](https://geocode.maps.co) on a cache miss, which needs the
  `GEOCODE_API_KEY` *repository* secret.

  **The cache is keyed by PLACE STRING, not by champion** — 422 champions live
  in 252 places, so renaming one costs nothing. It is committed because it is
  the only copy. The query is byte-identical to upstream's own
  `onetimeAddLocations.java`, so nobody visibly moves when source 1 takes over.

  Four behaviours are load-bearing:
  - **It never fails over geocoding.** No key, dead geocoder, exhausted quota:
    the run still writes every champion, just without new coordinates. A hard

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [foojayio/website](https://github.com/foojayio/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
