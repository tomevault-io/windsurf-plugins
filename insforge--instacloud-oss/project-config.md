---
trigger: always_on
description: For humans and agents alike. One template per folder under `templates/<code>/`, and the folder name
---

# Contributing a template

For humans and agents alike. One template per folder under `templates/<code>/`, and the folder name
IS the template code. Copying the closest existing template is the fastest way to start.

## Files

- `insta.template.yaml`: the manifest, and the source of truth. Required.
- `Dockerfile`: only when the template builds an overlay image. The workspace templates
  (`claude-code`, `codex`, `pi`) do; a template that references an official upstream image, like
  `n8n`, must not rebuild it. It is wired **by convention**: `templates-build-images.yml` builds
  `templates/<code>/` and pushes `ghcr.io/insforge/insta-oss/templates/<code>:<version>`, which the
  manifest then references as `image:`. Never add a `build:` key to the manifest. The catalog
  rejects a service carrying both `image:` and `build:`, and `image:` is the one that deploys.
- **Companion images.** A template directory builds exactly one image, so a template that needs a
  second one (a service built from source alongside the first, such as the browser worker `openmuse`
  runs next to its API, the way upstream's blueprint splits them) puts that image in its own sibling
  directory and references it. A service's `image:` may name a sibling template's published image,
  `ghcr.io/insforge/insta-oss/templates/<sibling>:<sibling-version>`, as long as that sibling exists
  here and the tag is its version; lint checks both. The sibling is usually `meta.draft: true`, so it
  never shows in the gallery on its own. A version bump moves a template's own image and leaves a
  companion alone: the companion moves when its own directory bumps.
- `README.md`: the detail page shown in the gallery. Required, and factual: leave a fact out rather
  than guess it. Follow the section order the existing templates use, which is Overview, what you
  get by hosting it, what you need before deploying, Configuration (a row per variable saying what
  it does and where the value comes from), After deploy (how to actually start using it), and
  Links (upstream, the image or package, the license). A draft template opens with a note saying
  why it is draft.
- The **deploy button**, in its own paragraph under the title and tagline, for every publishable
  template: `[![Deploy on InstaCloud](<cdn>/assets/deploy-button.svg)](https://instacloud.com/templates/<code>)`.
  CI rejects a publishable template that omits it, checks that the href names this template's own
  code, and rejects one on a draft, whose gallery page does not exist until it publishes. Since
  copying the nearest template is the fastest way to start, a code carried over from the one you
  copied is the mistake that check exists for. The publish step removes the line on the way to
  the catalog, so the gallery does not show a second copy of the call to action its own rail
  already carries. The asset and the snippet live in [assets/README.md](../assets/README.md).
- `logo.svg`: the template's mark, and a **hard requirement for a publishable template**. CI
  rejects a non-draft template without one. Declare the path as `meta.logo: ./logo.svg` and CI
  checks that it resolves. Details and the reasoning are under [Logos](#logos).
- Screenshots and any other README assets: keep them in your own template directory and reference
  them with **relative** paths, such as `![](./screenshot.png)`. The README is published to the
  gallery as text, where a relative path would resolve against the gallery's own origin and 404, so
  the publish step rewrites relative targets into absolute URLs pinned to the publishing commit:
  images through the same CDN as the logo, and links to the GitHub page a reader can browse. Two
  rules follow, both enforced by CI: a referenced asset must exist, and an image may not point
  outside its own template directory, because nothing outside it is the template's to ship.

## Hard rules (CI rejects violations)

1. Image references pin a specific tag or digest. `latest` or tagless is rejected, because an
   image that floats on `latest` silently changes under a deployed instance on every restart.
2. Every required variable without a generator has a `description` saying what it is and where to
   get it.
3. `code`, `version` (semver), `maintainer`, `upstream.pinned` and `meta.category` are mandatory.
4. A changed template must bump its `version`. The canonical image tag is derived from it, so
   editing a template without bumping would overwrite an image that published instances pull.
5. A template with a `Dockerfile` names its pin there too: `upstream.pinned` or `upstream.commit`
   has to appear in it, and where the `FROM` builds on `upstream.image`, that instruction's tag has
   to be the pinned one and its digest, if the manifest carries one, has to match. The version is
   written twice and nothing else notices when the two disagree: the image would be built from one
   version while the catalog advertises another, and the build would succeed.
6. A service may not carry both `image:` and `build:`.
7. `constraints[].oneOf` and `allOf` may only name variables the manifest declares.
8. A manifest never sizes a service. There is no `spec:`, and `volume:` is the boolean `true`, not

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [InsForge/instacloud-oss](https://github.com/InsForge/instacloud-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
