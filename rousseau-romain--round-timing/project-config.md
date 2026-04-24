---
trigger: always_on
description: RoundTiming is a Go web application for tracking game rounds/timing (likely for esports/gaming). It uses server-side rendering with Templ templates and TailwindCSS.
---

# Claude Code Instructions for RoundTiming

## Project Overview

RoundTiming is a Go web application for tracking game rounds/timing (likely for esports/gaming). It uses server-side rendering with Templ templates and TailwindCSS.

## Tech Stack

- **Go 1.25** with Gorilla Mux router
- **Templ** for HTML templating (`.templ` files compile to `_templ.go`)
- **TailwindCSS** for styling
- **MySQL** database via Docker
- **OAuth** authentication (Discord, Google) via Goth

## Development Commands

```bash
make live          # Start dev server with hot reload (localhost:7331)
make db_start      # Start database container
make db_stop       # Stop database container
make migration_up  # Run database migrations
make migration_down # Rollback last migration
make build/templ   # Generate templ files
make build/tailwind # Build CSS
```

## Project Structure

- `handlers/` - HTTP route handlers
- `routes/` - Route registration organized by domain:
  - `routes.go` - Main `Setup()` entry point
  - `public.go` - Static file serving
  - `pages.go` - Public pages (/, /privacy, /cgu, etc.)
  - `match.go` - All /match/* routes
  - `profile.go` - /profile/* and /user/*/locale/* routes
  - `auth.go` - /signup, /signin, /auth/* routes
  - `admin.go` - /admin/* routes
  - `errors.go` - /404, /403 routes
- `middleware/` - HTTP middleware (auth, language)
- `model/` - Database models and queries, organized by domain:
  - `model/db.go` - Shared database connection (`DB` exported for subpackages)
  - `model/user/` - User, UserSpectate, UserConfiguration, EmailWhiteListed
  - `model/match/` - Match, Team, Player, MatchPlayerSpell
  - `model/game/` - Class, Spell, SpellByClass (FavoriteSpell)
  - `model/system/` - Language, FeatureFlag
- `pkg/` - Reusable packages:
  - `password/` - Password hashing and validation
  - `lang/` - Language detection and supported languages
  - `constants/` - Application constants
  - `sqlhelper/` - SQL query helpers
- `views/page/` - Page templates (`.templ` files)
- `views/components/` - Reusable UI components organized by type:
  - `layout/` - Layout, Footer, Nav, PopinMessages
  - `ui/` - Buttons, AvatarToggle, ErrorPageContent
  - `icons/` - SVG icon components (Heart, User)
  - `forms/` - Form input components
- `service/auth/` - Authentication service (OAuth, session management)
- `config/` - Configuration loading
- `i18n/` - Internationalization

## Code Patterns

### Templ Templates

- Template files use `.templ` extension
- Run `templ generate` after modifying `.templ` files (or use `make live`)
- Generated files are `*_templ.go` - do not edit these directly
- Import components by package:
  ```go
  import "github.com/rousseau-romain/round-timing/views/components/layout"
  import "github.com/rousseau-romain/round-timing/views/components/ui"
  ```

### UI Components (`views/components/ui/`)

**Buttons** (`button.templ`):
```go
// Basic button
@ui.Button("primary", "md") { Click me }

// Button with HTMX attributes
@ui.ButtonAction("danger", "sm", templ.Attributes{
    "hx-delete": "/item/1",
}) { Delete }

// Link styled as button
@ui.ButtonLink("indigo", "lg", "/path") { Go }

// Link with custom attributes
@ui.ButtonLinkAction("black", "lg", "/path", templ.Attributes{
    "class": "w-full",
}) { Submit }
```

Variants: `primary` (blue), `success` (green), `danger` (red), `outline`, `indigo`, `black`
Sizes: `sm`, `md`, `lg`

**Badges** (`badge.templ`):
```go
// Generic badge with variant
@ui.Badge("red", templ.Attributes{"class": "absolute -right-1 -bottom-1"}) { 3 }

// Recovery badge (auto-colors based on rounds: 1=red, 2=yellow, 3+=cyan)
@ui.BadgeRecovery(mps.RoundBeforeRecovery)
```

Variants: `red`, `yellow`, `cyan`, `green`, `indigo`, `gray`

**Tables** (`table.templ`):
```go
@ui.Table("default") {
    @ui.TableHead("default") {
        <tr>
            @ui.Th() { Name }
            @ui.ThEmpty()
        </tr>
    }
    @ui.TableBody(templ.Attributes{
        "hx-swap": "outerHTML",
        "hx-target": "closest tr",
    }) {
        <tr>
            @ui.TdPrimary() { Item name }
            @ui.TdAction() { @ui.Button(...) }
        </tr>
    }
}
```

Table variants: `default` (full-width dividers), `compact` (bordered auto-width)
Rows: `Tr`, `TrBorder`, `TrColor(color)`
Header cells: `Th`, `ThEmpty`, `ThCompact`
Body cells: `Td`, `TdPrimary`, `TdCenter`, `TdAction`, `TdCompact`

**Containers & Cards** (`container.templ`):
```go
// Page container (centered with max-width)
@ui.Container("default") { ... }  // No padding
@ui.Container("padded") { ... }   // With p-4 padding

// Content card
@ui.Card("default") { ... }       // White bg with shadow
@ui.Card("bordered") { ... }      // With visible border

// Team-colored card (for match teams)
@ui.CardColor("red") { ... }      // border-red-200
@ui.CardColor("indigo") { ... }   // border-indigo-200

// Page section with margin
@ui.Section() { ... }

// Page title header
@ui.PageHeader() { Page Title }
```

### Form Components (`views/components/forms/`)

**forms.templ**:
```go
// Input with label (grid layout)
@forms.Input("email", "email", "email", "global.email", true)

// Flexible input with custom attributes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rousseau-romain) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
