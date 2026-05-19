---
trigger: always_on
description: You are an expert in TypeScript, Angular, and scalable web application development. You write maintainable, performant, and accessible code following Angular and TypeScript best practices.
---

You are an expert in TypeScript, Angular, and scalable web application development. You write maintainable, performant, and accessible code following Angular and TypeScript best practices.

This project is an Nostr project, that uses nostr-tools library. Make sure you follow the Nostr NIPs protocol definitions. Nostr uses
timestamp for dates that is in seconds, not milliseconds.

The application uses Angular Material, so make sure to use Angular Material components when possible.

Always use "fetch" for http request instead of HttpClient.

Never set the font-weight in CSS. The current font for headlines does not support different font weights.

For dialogs, don't use Angular Material dialogs, but the custom "CustomDialogComponent" component instead.
Never use native `confirm()` dialogs. Use app dialogs/snackbars for confirmation flows.

URL for this app is: https://nostria.app

## Architecture

To understand the architecture, see [ARCHITECTURE.md](../ARCHITECTURE.md). Important to follow the architecture decisions made there.

Whenever you are doing some architecture or implementation changes that are not trivial, make sure to update the architecture documentation.

## Command Palette

When adding new features or routes, always add corresponding commands to the Command Palette (`src/app/components/command-palette-dialog/`). This ensures users can access all features via keyboard shortcuts (Ctrl+K).

## TypeScript Best Practices

- Use strict type checking
- Prefer type inference when the type is obvious
- Avoid the `any` type; use `unknown` when type is uncertain

## Angular Best Practices

- Always use standalone components over NgModules
- Must NOT set `standalone: true` inside Angular decorators. It's the default.
- Use signals for state management
- Implement lazy loading for feature routes
- Do NOT use the `@HostBinding` and `@HostListener` decorators. Put host bindings inside the `host` object of the `@Component` or `@Directive` decorator instead
- Use `NgOptimizedImage` for all static images.
  - `NgOptimizedImage` does not work for inline base64 images.

## Components

- Keep components small and focused on a single responsibility
- Use `input()` and `output()` functions instead of decorators
- Use `computed()` for derived state
- Set `changeDetection: ChangeDetectionStrategy.OnPush` in `@Component` decorator
- Prefer inline templates for small components
- Prefer Reactive forms instead of Template-driven ones
- Do NOT use `ngClass`, use `class` bindings instead
- DO NOT use `ngStyle`, use `style` bindings instead
- DO NOT put color="primary" on buttons, this is not supported in Material 3. Also for primary actions, use "mat-flat-button", not "flat-raised-button".

## State Management

- Use signals for local component state
- Use `computed()` for derived state
- Keep state transformations pure and predictable
- Do NOT use `mutate` on signals, use `update` or `set` instead

## Templates

- Keep templates simple and avoid complex logic
- Use native control flow (`@if`, `@for`, `@switch`) instead of `*ngIf`, `*ngFor`, `*ngSwitch`
- Use the async pipe to handle observables

## Services

- Design services around a single responsibility
- Use the `providedIn: 'root'` option for singleton services
- Use the `inject()` function instead of constructor injection

## Styling

Always use "field-sizing: content" for textareas that grow with content. This is compatible with all modern browsers.

When resizing `mat-icon-button` smaller than its default size, never use `line-height` to center the icon. Instead use `padding: 0 !important; display: flex !important; align-items: center; justify-content: center;` to properly center the icon within the button.

- The app supports dark and light mode, so make sure your styles work well in both modes.
- Don't add hardcoded colors. Use CSS variables defined in styles.scss and theme.scss

Due to Angular component styles are encapsulated by default, so use this way to ensure dark mode is applied correctly:

```css
:host-context(.dark) .your-class {
  background-color: var(--mat-sys-surface-container);
  color: var(--mat-sys-on-surface);
}
```

Don't make documentation for every change, only important and hard to understand fixes.

When you generate markdown documentation of what you have done, place those documents into the "docs" folder.

## TODO: The color scheme will change, so don't rely on the colors, just the variables!

These are the CSS variables for Angular Material 3, don't use old variables.

    --mat-success-color: #66bb6a;
    --mat-success-lighter: #a5d6a7;
    --mat-success-darker: #388e3c;
    --scrollbar-track: #424242;
    --scrollbar-thumb: #686868;
    --scrollbar-thumb-hover: #7e7e7e;
    --mat-sys-background: #18111b;
    --mat-sys-error: #ffb4ab;
    --mat-sys-error-container: #93000a;
    --mat-sys-inverse-on-surface: #362e39;
    --mat-sys-inverse-primary: #5953a9;
    --mat-sys-inverse-surface: #ecdeed;
    --mat-sys-on-background: #ecdeed;
    --mat-sys-on-error: #690005;
    --mat-sys-on-error-container: #ffdad6;
    --mat-sys-on-primary: #2a2278;
    --mat-sys-on-primary-container: #e3dfff;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nostria-app/nostria](https://github.com/nostria-app/nostria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
