---
trigger: always_on
description: This is the browser extension (Chrome MV3) for PearPass. It's written in React + TypeScript. The extension has four build targets — the action popup ([src/action/](src/action/)), content scripts ([src/content/](src/content/)) and their in-page popups ([src/contentPopups/](src/contentPopups/)), an injected page script ([src/inject/](src/inject/)), and the background service worker ([src/background/](src/background/)). UI is built on the shared component library `@tetherto/pearpass-lib-ui-kit`.
---

# UI conventions for pearpass-app-browser-extension

This is the browser extension (Chrome MV3) for PearPass. It's written in React + TypeScript. The extension has four build targets — the action popup ([src/action/](src/action/)), content scripts ([src/content/](src/content/)) and their in-page popups ([src/contentPopups/](src/contentPopups/)), an injected page script ([src/inject/](src/inject/)), and the background service worker ([src/background/](src/background/)). UI is built on the shared component library `@tetherto/pearpass-lib-ui-kit`.

This document is for **anyone contributing UI** to the repo — new hires, current engineers, and AI coding assistants (Claude Code, Cursor, Codex, etc.). It captures the component catalog, styling conventions, file-naming rules, and patterns we use when building UI in this extension. Read it once before your first UI change; keep it open when you're in doubt.

## Migration state — v2 is the target

UI is being migrated onto `@tetherto/pearpass-lib-ui-kit`. For **new UI work** — v2 redesigns of existing screens and net-new features — use the kit. Today the only kit wiring in this repo is the `ThemeProvider` in [src/action/index.jsx](src/action/index.jsx); every actual component rendered in the UI still comes from the legacy tree at [src/shared/components/](src/shared/components/). That tree keeps working while the migration is in progress — do not delete anything from it as part of v2 work.

Note: unlike the sibling desktop repo, this extension does **not** currently gate v1 vs v2 behind a `EXTENSION_DESIGN_VERSION` / `isV2()` runtime flag. Kit-based components render directly at their call sites. If a flag is added later, this doc will describe it.

## File naming: when to use the `V2` suffix

The `V2` suffix is a **coexistence marker**, not a design marker. Use it only when a v1 sibling already exists:

- **A v1 file already exists** for this component/screen → create a new file with the `V2` suffix next to it (e.g. v1 `CreateVaultModalContent.jsx` → new `CreateVaultModalContentV2.tsx`). Both live in the tree during migration; the branching happens at the call site.
- **No v1 equivalent exists** (net-new feature, net-new component) → create the file with its natural name, **no `V2` suffix**. The suffix would just be noise.

Before creating a file, glob the directory for the base name without the suffix. If nothing comes up, skip the suffix.

## Where UI lives in this repo

- [src/action/pages/](src/action/pages/) — top-level screens for the action popup (WelcomePage, Settings, RecordDetails, RecordList, CreatePasskey, SelectPasskey, AuthenticatorView, AddDevice, NonSecureWarning).
- [src/action/containers/](src/action/containers/) — stateful composed UI used by those pages.
- [src/contentPopups/views/](src/contentPopups/views/) — in-page popups rendered by the content script (Autofill, LoginDetect, PasswordGenerator, PasswordSuggestion, Logo).
- [src/shared/components/](src/shared/components/) — **legacy v1 custom components** (do not grow; see rules below).
- [src/shared/containers/](src/shared/containers/) — shared stateful UI (Sidebar, PassPhrase, CreateVaultModalContent, etc.).

## Golden rules

1. **Check the catalog below before creating any component.** If it exists in the kit, use it — never wrap or reimplement.
2. **All new UI goes through the kit.** Any new `.tsx`/`.jsx` file — suffixed or not — must import from `@tetherto/pearpass-lib-ui-kit`, not from [src/shared/components/](src/shared/components/).
3. **Never add variants under [src/shared/components/](src/shared/components/)** (`ButtonPrimary`, `ButtonSecondary`, `ButtonRoundIcon`, `ButtonFilter`, `ButtonFolder`, `ButtonCreate`, `ButtonLittle`, `ButtonSingleInput`, `ButtonPlusCreateNew`, `InputPearPass`, `InputPasswordPearPass`, `InputField`, `InputFieldPassword`, `InputSearch`, etc.). That tree is legacy; the kit's `Button` takes variants.
4. **Style with Tailwind utilities mapped to kit tokens** — `bg-surface-primary`, `text-text-primary`, `border-border-primary`, etc. Use `useTheme()` / `rawTokens` only when a value is needed in JS. No hardcoded hex colors or px spacing for design-system values.
5. **Icons come from the kit.** `@tetherto/pearpass-lib-ui-kit/icons` has 133 icons. Do not add new SVGs under `src/`.
6. **If the kit lacks something you need, stop and ask the user.** Don't silently roll a custom component.

## Component catalog (33 components)

Import pattern: `import { ComponentName } from '@tetherto/pearpass-lib-ui-kit'`

### Actions
- `Button` — all CTAs. Takes variants; use instead of `ButtonPrimary`, `ButtonSecondary`, `ButtonRoundIcon`, `ButtonLittle`, `ButtonFilter`, `ButtonFolder`, `ButtonSingleInput`, `ButtonCreate`, `ButtonPlusCreateNew`.
- `Pressable` — low-level pressable wrapper for custom interactive elements.
- `Link` — text links.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tetherto/pearpass-app-browser-extension](https://github.com/tetherto/pearpass-app-browser-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
