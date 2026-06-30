---
trigger: always_on
description: Open Lap is a slot car race management app for Carrera® DIGITAL 124/132 systems. To use Open Lap, compatible hardware such as the [Carrera Bluetooth Connect](https://carrera-toys.com/en-at/products/20030369-carrera-bluetooth-connect) is required to connect to the track's Control Unit.
---

# Copilot Instructions for Open Lap

## Project Overview



Open Lap is a slot car race management app for Carrera® DIGITAL 124/132 systems. To use Open Lap, compatible hardware such as the [Carrera Bluetooth Connect](https://carrera-toys.com/en-at/products/20030369-carrera-bluetooth-connect) is required to connect to the track's Control Unit.

The app is built with [Ionic 8](https://ionicframework.com/), [Angular 20](https://angular.io/), and [Apache Cordova](https://cordova.apache.org/) for native mobile builds. It also runs as a Progressive Web App.


### Related Software

Open Lap is an alternative to other slot car management apps:
- [Carrera RaceApp](https://carrera-toys.com/en-at/pages/landing-page-race-app): Official app, no longer maintained, limited features.
- [SmartRace](https://www.smartrace.de/en/): Commercial app with more features, but not free.

Open Lap aims to provide a free, open-source solution for race management, focusing on usability and extensibility.

## Architecture

- **NgModule-based** architecture throughout — no standalone components.
  All components set `standalone: false` explicitly.
- Feature modules: `BackendModule`, `DriversModule`, `MenuModule`, `RmsModule`,
  `SharedModule`, `TuningModule`, with `SettingsModule` lazy-loaded.
- Sub-feature modules exist within features (e.g., `LeaderboardModule`,
  `RaceControlModule` inside `rms/`) — imported by their parent module.
- Every feature directory has an `index.ts` barrel file; use short barrel imports
  (e.g., `'../services'`, `'../carrera'`).
- No state management library — settings are managed via `SettingsService`
  (wrapping `@ionic/storage-angular`) with an `AppSettings` typed facade using
  Observables.

### Backend Pattern

- Abstract `Backend` class with `abstract scan(): Observable<Peripheral>`.
- Concrete implementations are `@Injectable()` without `providedIn` and
  registered via multi-provider DI in `BackendModule`:
  `{ provide: Backend, useClass: SomeBackend, multi: true }`.
- Injected as an array: `@Inject(Backend) private backends: Backend[]`.

### Domain Model (`src/app/carrera/`)

- Pure TypeScript classes/interfaces with no Angular decorators.
- Heavy RxJS usage (`BehaviorSubject`, `Observable`, `Subject`, `Connectable`).
- `ControlUnit` is a plain class (not injectable), instantiated directly.
- Custom `DataView` class shadows the native browser `DataView` — be careful
  with imports.
- `Session` class in `rms/` is also a non-injectable domain class.

## Code Style

- **Single quotes** for all strings and imports.
- **Semicolons** always.
- **2-space indentation**.
- No linter or formatter configured — style is convention-based.
- TypeScript strict mode with `strictNullChecks: false` and `noImplicitAny: false`.
- `useDefineForClassFields: false` in tsconfig.
- Templates in separate HTML files (`templateUrl`), styles in separate SCSS files
  (`styleUrls`). Some pages omit `styleUrls` when no styling is needed.
- Use legacy Angular template syntax (`*ngIf`, `*ngFor`, `[ngSwitch]`) — not
  the newer `@if`/`@for` control flow.

## Naming Conventions

| Item             | Pattern                    | Example                     |
| ---------------- | -------------------------- | --------------------------- |
| Components       | `kebab-case.component.ts`  | `checkerboard.component.ts` |
| Pages            | `kebab-case.page.ts`       | `rms.page.ts`               |
| Services         | `kebab-case.service.ts`    | `settings.service.ts`       |
| Modules          | `kebab-case.module.ts`     | `rms.module.ts`             |
| Pipes            | `kebab-case.pipe.ts`       | `time.pipe.ts`              |
| Directives       | `kebab-case.directive.ts`  | `target.directive.ts`       |
| Popover menus    | `kebab-case.menu.ts`       | `rms.menu.ts`               |
| Domain models    | `kebab-case.ts` (no suffix)| `control-unit.ts`           |

## Components vs Pages

- **Pages** are routed to and have no `selector` property.
- **Components** used in templates have a `selector` (typically unprefixed
  lowercase, e.g., `checkerboard`, `gauge`, `connections`).
- **Popover menus** (`*.menu.ts`) receive data via `@Input()` properties
  (passed as `componentProps`) and use `PopoverController` to dismiss.
- **Modal dialogs** use `ModalController`, receive data via `@Input()`
  properties (passed as `componentProps`), and dismiss with data.

## Services

- Most services use `@Injectable({ providedIn: 'root' })`.
- `ControlUnitService` **extends `BehaviorSubject<ControlUnit>`** — components
  pipe from it directly as an Observable source.
- Cordova plugin wrappers are provided in the root module's `providers` array.
- Settings subscription pattern: subscribe in `ngOnInit`, store as
  `private subscription: Subscription`, unsubscribe in `ngOnDestroy`.
  Some components use `firstValueFrom()` instead.
- Pipes carry both `@Pipe()` and `@Injectable()` decorators.

## Internationalization

- Uses **@ngx-translate** (v17 functional providers: `provideTranslateService()`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tkem/openlap](https://github.com/tkem/openlap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
