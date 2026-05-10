---
trigger: always_on
description: - The public starting point for Snaps should be the docs-site homepage, not the repo. It should be a stable, friendly URL that humans and agents can both begin from.
---

# snaps

## Philosophy and Design Principles

- The public starting point for Snaps should be the docs-site homepage, not the repo. It should be a stable, friendly URL that humans and agents can both begin from.
- The docs site should be creator-first by default: optimize the homepage and primary navigation for people who are curious about Snaps or want to create one, while still making technical integration and agent paths easy to discover.
- Repo-native docs should be integrator-first: assume the reader is a developer working on the spec/code itself, building on top of it, or integrating Snaps into their own product.
- The audience split should be visible in the information architecture. Do not hide major distinctions like learn, create, integrate, and agents purely in folder layout or internal structure.
- Prefer progressive disclosure over dumping everything in one place. Top-level entrypoints should orient and route; deeper pages should hold the detail.
- Every important fact should have one primary home. Other surfaces should point to that home instead of restating the same information in slightly different words.
- Treat documentation as a set of surfaces, not just files: docs-site pages, repo markdown, skills, code comments, GitHub rendering, machine-readable exports, and npm package pages all shape how people and agents understand the system.
- Code and schemas are canonical for protocol behavior. Prose documentation should explain, organize, and point back to the implementation truth in `pkgs/snap`, not compete with it.
- Documentation work should not change the meaning of the system. Reorganize, split, move, and clarify docs freely, but do not silently change protocol semantics, validation behavior, or implementation behavior.
- If docs are inconsistent, preserve the canonical behavior and call out the discrepancy explicitly rather than “fixing” it by changing meaning in prose.
- Keep workflow docs and reference docs distinct. Skills and README files should mainly help people and agents find the right path and execute tasks; they should not become shadow specs.
- Use `AGENTS.md` to capture intent, philosophy, and high-level design direction so agents can understand where the project is going, not just how it works today.

## Operational handbook

Ports, env vars, package behaviors, build quirks, and other workspace-specific facts live in [`WORKSPACE.md`](./WORKSPACE.md). Read that file when you need a specific operational detail.

## Learned User Preferences

- **Deep modules principle:** Each workspace package under `pkgs/` keeps a single public surface: `src/index.ts` must list that module’s exports. **All imports from outside the module go through that entry only** (package name such as `@farcaster/snap`, which resolve to `index.ts`). Do not import another package by reaching into its implementation paths (for example `…/snap/src/server/hubs` via relative file paths from a different package). Inside the same package, implementation files import each other with relative paths; do not re-export through random barrels—only `index.ts` aggregates what other packages may use. Do not use `export { … } from "./other"` or `export * from "./other"` in non-`index` implementation files as a shortcut for external consumers; add exports to `index.ts` instead.
- Do not use the `any` type in `pkgs/snap/src/schemas.ts` or `pkgs/snap/src/validator.ts`. Avoid `unknown` there except where required for external or untyped input (for example the top-level value passed into validation entrypoints). Prefer concrete types everywhere else.
- Prefer exporting shared string and enum values as constants from `pkgs/snap/src/schemas.ts` when they are used in schemas or validation logic.
- Validation errors use Zod’s issue `code` and `message` (plus `path`); there is no separate FC error-code layer or `errors.ts` barrel.
- Keep normative snap spec text in the docs app MDX under `apps/docs/src/app/(docs)/` — home entrypoints (`(home)/page.mdx`, `(home)/agents/page.mdx`), learn guides (`(learn)/*/page.mdx`), and snap spec pages (`(spec)/*/page.mdx`) — instead of maintaining a separate spec tree.
- **Skill docs:** Prefer linking to the docs site (or [docs.farcaster.xyz/snap](https://docs.farcaster.xyz/snap)) instead of restating spec rules. Keep the snap skill deployment-first and template-driven (point to `template/` as source of truth). The primary “create a snap from a prompt” skill is inlined in `apps/docs/src/app/SKILL.md/route.ts`, served at https://docs.farcaster.xyz/snap/SKILL.md.
- For server-generated snap previews (for example Open Graph images in `@farcaster/snap-hono`), prefer font files shipped with the app (for example under the deployment template’s static assets) and paths passed through handler options over relying only on runtime CDN font fetches, so Vercel and edge deploys stay predictable. **Satori** (used for OG PNG generation in that path) loads **TTF, OTF, or WOFF (v1)** only; **WOFF2 is not supported**—use **WOFF or TTF** font files when embedding fonts for OG.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [farcasterxyz/snap](https://github.com/farcasterxyz/snap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
