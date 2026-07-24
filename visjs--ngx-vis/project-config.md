---
trigger: always_on
description: Angular library (Angular 21) wrapping [vis.js](https://visjs.org/) network and timeline components. Published via **ng-packagr**. Contains a standalone demo app in `demo/`.
---

# ngx-vis — Claude Code Context

## Project overview

Angular library (Angular 21) wrapping [vis.js](https://visjs.org/) network and timeline components. Published via **ng-packagr**. Contains a standalone demo app in `demo/`.

- **Library source**: `components/` + entry point `ngx-vis.ts`
- **Demo app**: `demo/` (standalone Angular app, separate `package.json`)
- **Built output**: `dist/` (ng-packagr), `docs/` (GitHub Pages build)
- **Tests**: Jasmine unit tests in `components/**/*.spec.ts`, run via Angular CLI karma builder

---

## Key versions

| Package | Version |
| --- | --- |
| Angular | ^21.2.0 |
| TypeScript | ~5.9.0 (ng-packagr 21 requires >=5.9) |
| zone.js | ~0.16.0 |
| rxjs | ~7.8.0 |
| ng-packagr | ^21.2.0 |
| Node.js | >=20.11 |

---

## Repository structure

```text
ngx-vis/
├── components/
│   ├── network/
│   │   ├── vis-network.directive.ts
│   │   ├── vis-network.directive.spec.ts
│   │   ├── vis-network.service.ts
│   │   └── vis-network.service.spec.ts
│   └── timeline/
│       ├── vis-timeline.directive.ts
│       ├── vis-timeline.service.ts
│       └── vis-timeline.service.spec.ts
├── demo/                        # standalone Angular app (own package.json)
│   ├── angular.json
│   ├── package.json
│   ├── tsconfig.json            # moduleResolution: bundler
│   └── src/
│       ├── main.ts              # bootstrapApplication (standalone, no AppModule)
│       └── app/
│           ├── app.component.ts       # imports: [RouterOutlet, RouterLink]
│           ├── app.routes.ts          # route definitions
│           ├── network/network-example.component.ts  # imports: [NgIf, VisModule]
│           └── timeline/timeline-example.component.ts # imports: [VisModule]
├── .config/                     # legacy webpack configs (not used for main builds)
├── angular.json                 # root-level: library test target only
├── karma.conf.js                # sets CHROME_BIN via puppeteer, coverage reporter
├── ng-package.json
├── ngx-vis.ts                   # library entry: exports + VisModule NgModule
├── tsconfig.json                # root: target ES2022, module commonjs (for webpack compat)
├── tsconfig.publish.json        # ng-packagr compile: module ES2022, target ES2022
├── tsconfig.spec.json           # test compile: extends root, adds importHelpers, bundler resolution
└── package.json
```

---

## NPM scripts

| Script | What it does |
| --- | --- |
| `npm run build` | Build library via ng-packagr → `dist/` |
| `npm test` | Run unit tests (ChromeHeadless via puppeteer) with coverage |
| `npm run test:watch` | Same but keeps watching |
| `npm run flow.build:demo` | Pack library, install in demo, `ng build` demo |
| `npm run build:docs` | Clean `docs/`, build GitHub Pages output |
| `npm run flow.build:github-pages` | Pack library → install in demo → `ng build --base-href=/ngx-vis/ --output-path=../docs` |
| `npm run clean` | Remove build artefacts |
| `npm run flow.lint` | ESLint only (tslint removed) |

---

## Architecture decisions

### Library: standalone directives + VisModule

Angular 19+ makes directives **standalone by default**. `VisNetworkDirective` and `VisTimelineDirective` are standalone. `VisModule` is kept as a backward-compatible convenience NgModule that **imports** (not declares) them:

```typescript
@NgModule({
  imports: [CommonModule, VisNetworkDirective, VisTimelineDirective],
  exports: [VisNetworkDirective, VisTimelineDirective],
  providers: [VisNetworkService, VisTimelineService],
})
export class VisModule {}
```

Consumer components can either `import: [VisModule]` (NgModule style) or import the directives directly.

### Demo: fully standalone (no AppModule)

`demo/src/main.ts` uses `bootstrapApplication`. There is no `AppModule` or `AppRoutingModule`. Routes live in `demo/src/app/app.routes.ts`. Each component has its own `imports` array.

### Demo builder: `application` (esbuild)

`demo/angular.json` uses `@angular-devkit/build-angular:application` (required from Angular 20+; the old `browser` builder was removed). Key rename: `main` → `browser` in options. Dev server uses `buildTarget` instead of `browserTarget`.

### Build budgets (demo)

Raised to `1.5mb` warning / `3mb` error because vis.js ships only CJS modules and cannot be tree-shaken, resulting in a larger-than-default initial bundle.

---

## Testing

**Always write unit tests** when implementing new features or fixing bugs. Every change to a service or directive must be accompanied by a corresponding spec file update or addition.

Tests are in `components/**/*.spec.ts`. They are **pure unit tests** — no `TestBed`, services and directives are instantiated directly. `VisTimelineService` receives an NgZone mock. Directives are tested by instantiating them with a mock `ElementRef` and a `jasmine.createSpyObj` for the service dependency.

### Patterns

- **Services**: instantiate directly, call methods, assert return values or thrown errors.
- **Directives**: instantiate directly with `new ElementRef(document.createElement('div'))` and a spy service; call lifecycle hooks (`ngOnInit`, `ngOnChanges`, `ngOnDestroy`) manually and assert spy calls.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [visjs/ngx-vis](https://github.com/visjs/ngx-vis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
