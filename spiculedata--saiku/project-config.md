---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build, test, run

JDK 21 + Maven 3.9+ are required. The reactor is `saiku-bom → saiku-core → saiku-webapp → saiku-launcher`. `saiku-ui` is a separate SvelteKit app and is **not** a reactor module — but `mvn verify` still builds it, via `saiku-webapp`'s frontend plugin (see *Testing baseline*), so a cold build downloads a Node toolchain.

**On Windows, set `git config core.autocrlf false` and re-clone before building.** Git for Windows defaults `core.autocrlf=true`, and the repo has no `.gitattributes` to override it, so checkout rewrites every LF blob to CRLF and `spotless:check` then rejects every Java file — before anything compiles. CI never catches this because CI is Linux-only.

```bash
mvn verify                                    # compile + unit tests + spotless:check (CI's gate)
mvn spotless:apply                            # auto-format Java (Palantir Java Format, bound in root pom)
mvn -pl saiku-core/saiku-service -am test     # one module's tests
mvn -pl saiku-core/saiku-service test -Dtest=RepositoryDatasourceManagerTest  # single test class
mvn -pl saiku-webapp,saiku-launcher clean                     # WIPE stale UI bundles before rebuilding — CRITICAL for UI changes
mvn -pl saiku-launcher -am -Dmaven.test.skip=true package    # build the runnable fat-JAR
mvn -P security verify                        # OWASP dependency-check (opt-in)
./scripts/install-hooks.sh                    # one-time: install pre-commit spotless hook
```

Run the launcher fat-JAR (Picocli + embedded Jetty 12 EE10):

```bash
java -jar saiku-launcher/target/saiku-<version>.jar serve --port 8080 --home ./saiku-home
# <version> is the root pom's <version> — `mvn -q -DforceStdout help:evaluate -Dexpression=project.version`
# UI:    http://localhost:8080/ui/
# REST:  http://localhost:8080/saiku/api/...
# Login: admin / admin
```

For the SvelteKit UI (`saiku-ui/`, version-tracked independently as 3.17.0):

```bash
cd saiku-ui && npm install && npm run dev     # vite dev server
npm run check                                 # svelte-check + tsc
npm test                                      # vitest
npm run lint                                  # ESLint flat config (token-only rule)
npm run storybook                             # Storybook 10.4 design-system catalogue
npm run build                                 # static build → saiku-ui/dist
```

The UI is **Tailwind v4 + design-system primitives + Storybook**. Token bridge at `src/lib/styles/tailwind.css` uses `@theme inline` to map the 169 saiku-ui tokens (in `src/lib/styles/tokens.css`) onto Tailwind's namespace; primitives live at `src/lib/components/ui/` (shadcn-style with `tailwind-variants` + `bits-ui` Tooltip) and reusable widgets at `src/lib/design-system/` (14 primitives + 19 stories). ESLint bans raw tone classes (`bg-emerald-*`, `text-red-*`, `bg-amber-*`, `rose`, `orange`) outside `src/lib/design-system/` — token utilities only.

**Cascade-layer discipline (load-bearing):** every type-selector rule in `app.css` MUST live in `@layer base` so Tailwind utilities (in `@layer utilities`) can override them. Unlayered rules win against ALL layered rules regardless of specificity — that's the CSS cascade-layers spec, and the saiku-ui legacy `a { color: var(--accent) }` ate `text-primary-foreground` on every anchor-as-button until the wrap landed. Only the `*, *::before, *::after { box-sizing }` reset and the `html, body { height/margin }` root zeroing stay unlayered.

**Stale-bundle gotcha with the launcher fat-JAR (bit us twice):** `saiku-webapp`'s maven-war-plugin overlays `saiku-ui/dist/` into `webapp/saiku.war` inside the launcher fat-JAR, but it does NOT purge stale files from `target/`. SvelteKit generates hashed filenames like `_app/immutable/nodes/2.<hash>.js` — every `npm run build` produces new hashes, and every `mvn package` that skips `clean` leaves the old hashes AND the new hashes both in the war. The server's `index.html` points at the new hashes; the old bundle files sit there dead. But if you're debugging via `unzip -l` you can miss which one is live. Fix: `mvn -pl saiku-webapp,saiku-launcher clean` before any UI rebuild — never trust an incremental package on top of an existing `target/` when saiku-ui bundles have changed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spiculedata/saiku](https://github.com/spiculedata/saiku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
