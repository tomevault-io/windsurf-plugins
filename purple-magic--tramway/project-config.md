---
trigger: always_on
description: This document guides AI-assisted code generation for Rails applications built with **Tramway**. It is designed for founders, designers, PMs, and engineers who rely on AI tools (ChatGPT/Copilot/etc.) to build maintainable Rails code that fits Tramway’s conventions without a senior engineer reworking everything.
---

## Start of Tramway AGENTS.md

# AGENTS.md: Tramway Code Generation Standards

This document guides AI-assisted code generation for Rails applications built with **Tramway**. It is designed for founders, designers, PMs, and engineers who rely on AI tools (ChatGPT/Copilot/etc.) to build maintainable Rails code that fits Tramway’s conventions without a senior engineer reworking everything.

**Core Principle:** Generated code should feel like curated documentation—simple, explicit, and aligned with Tramway’s defaults. Prefer composable ViewComponents, Tailwind-friendly markup, and Rails-native patterns over bespoke architectures.

---

## Project Overview

Tramway extends Rails with:
- **CRUD** actions that can be configured in `config/initializers/tramway.rb`.
- **Generators** that wire Tailwind, ViewComponent, and pagination defaults (`bin/rails g tramway:install`).
- **ViewComponents** for reusable UI pieces.
- **Tailwind safelist** utilities to keep dynamic classes in the build.

Generated code should:
- Lean on Rails conventions and Tramway generators instead of hand-rolled setup.
- Use domain language (e.g., `Participant`, `Dashboard`) over generic terms.
- Keep logic in the right layer: models for data/validations, controllers for HTTP, components for repeatable UI, views for simple rendering.
- Be readable without comments; short methods, guard clauses, and clear naming.

---

## Quick Start Workflow (Preferred)

1) **Install Tramway defaults**

```bash
bin/rails g tramway:install
```

- The install generator appends missing gems, copies Tailwind safelist config, ensures `app/assets/tailwind/application.css` imports Tailwind, and adds a Codex instruction to use or install `tramway-skill`.

---

## Technology Stack & Gems

Tramway expects and installs:
- `rails` (7+), `kaminari`, `view_component`, `haml-rails`, `dry-initializer`, `tailwindcss-rails`.
- Prefer Haml for views unless a component template uses ERB.
- Keep JavaScript minimal; use Stimulus if needed, avoid SPAs.

Do **not** introduce alternative architectures (contexts/operation gems) unless explicitly requested.

---

## File Structure & Organization

Follow Rails defaults. When extracting logic, namespace under the owning model or component:

```
app/
  components/               # ViewComponent classes and templates
    participants/
      card_component.rb
      card_component.html.erb
  controllers/
    participants_controller.rb
  decorators/               # Tramway Decorator pattern
    participant_decorator.rb
  forms/                    # Tramway Form pattern
    participant_form.rb
  models/
    participant.rb
  views/
    participants/
      show.html.haml
config/
  initializers/
      tramway.rb              # Tramway configuration
  tailwind.config.js        # Safelist managed by tramway:install
```

---

## Rules

### Rule 1
If CRUD is requested or some default actions like (index, show, create, update, destroy) are requestsed, use Tramway Entities by default unless custom behavior is needed. Configure in `config/initializers/tramway.rb`. Do not create controllers, views, or routes manually for CRUD actions if Tramway Entities can handle it.

When `namespace` is mentioned in the request, configure it in the entity definition.

Example of CRUD configuration for model `Participant`:

*config/initializers/tramway.rb*:
```ruby
Tramway.configure do |config|
  config.entities = [
    {
      name: :participant,
      pages: [
        { action: :index },
        { action: :show },
        { action: :create },
        { action: :update },
        { action: :destroy }
      ]
    }
  ]
end
```

If admin panel requested to be implemented from scratch, do the same with `namespace: :admin`

### Rule 1.1
Search is disabled by default on index pages. Enable it with `search: true` on the `:index` page definition:

```ruby
Tramway.configure do |config|
  config.entities = [
    {
      name: :participant,
      pages: [
        {
          action: :index,
          search: true
        }
      ]
    }
  ]
end
```

If `Model.search` exists, Tramway uses it. Otherwise it falls back to `Model.tramway_search` and logs a warning.
The fallback is generic, not tailored to the data structure, and should not be used long-term because it may be slow or not scalable.

### Rule 2
Normalize input with `normalizes` (from Tramway) for attributes like email, phone, etc. Don't use `normalizes` in model unless it requested explicitly.

### Rule 2.1
When you need form-level validation, use Tramway Form `validates` on the form object (ActiveModel/ActiveRecord validation options like `presence: true` work, and `with:` is optional unless a validator requires it). Keep data integrity validations in the model unless the request explicitly needs form-only logic.

### Rule 3
Use Tramway Navbar for navigation. Put there basic links: Login, Logout.

### Rule 4
Use Tramway Flash for user notifications.

```
= tramway_flash text: flash[:notice], type: :hope

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Purple-Magic/tramway](https://github.com/Purple-Magic/tramway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
