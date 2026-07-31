---
trigger: always_on
description: Backpex is a highly customizable administration panel for Phoenix LiveView applications. It allows you to quickly create CRUD views of your existing data using configurable LiveResources. Backpex integrates seamlessly with your existing Phoenix LiveView application and provides an easy way to manage your resources. It is highly customizable and can be extended with your own layouts, views, field types, filters and more.
---

# Context

## Project Overview

Backpex is a highly customizable administration panel for Phoenix LiveView applications. It allows you to quickly create CRUD views of your existing data using configurable LiveResources. Backpex integrates seamlessly with your existing Phoenix LiveView application and provides an easy way to manage your resources. It is highly customizable and can be extended with your own layouts, views, field types, filters and more.

**Key Features**:
- **LiveResources**: Quickly create LiveResource modules for your database tables with fully customizable CRUD views. Bring your own layout or use our components.
- **Search and Filters**: Define searchable fields and add custom filters for instant, LiveView-powered results.
- **Resource Actions**: Implement global custom actions like user invitations or exports, with support for additional form fields.
- **Authorization**: Handle CRUD and custom action authorization via simple pattern matching, with optional integration for external authorization libraries.
- **Field Types**: Out-of-the-box support for Text, Number, Date, Upload, and more. Easily extend with your own custom field type modules.
- **Associations**: Effortlessly handle HasOne, BelongsTo, and HasMany(Through) associations with minimal configuration. Customize available options and rendered columns.
- **Metrics**: Add value metrics such as sums or averages for quick data insights, with more metric types on the horizon.

## Tech Stack

### Programming Language

- **Elixir**: Functional programming language for building scalable and maintainable applications

### Framework

- **Phoenix LiveView**: Real-time web framework for Elixir

### Styling

- **Tailwind CSS**: utility-first CSS framework
- **daisyUI**: Tailwind CSS plugin and component library

## Architecture

Backpex follows a modular architecture that separates the library code from the demonstration application. The project is structured as an Elixir library that can be integrated into Phoenix LiveView applications.

### Project Structure

The repository contains two main parts:

**1. Backpex Library (`/`)**

The root directory contains the Backpex library itself, which is published as a Hex package. Key directories:

```
lib/backpex/
├── adapters/          # Data layer adapters (Ecto)
├── controllers/       # Phoenix controllers (cookie management)
├── fields/            # Built-in field types (Text, Number, Date, BelongsTo, HasMany, etc.)
├── filters/           # Built-in filter types (Boolean, Select, Range, etc.)
├── html/              # Phoenix Components for UI rendering
│   ├── core_components.ex    # Base UI components
│   ├── layout.ex              # Layout components
│   ├── form.ex                # Form components
│   └── resource.ex            # Resource-specific components
├── item_actions/      # Built-in item actions (Edit, Delete, Show)
├── live_components/   # LiveView components
├── live_resource/     # Core LiveResource views (Index, Form, Show)
├── metrics/           # Metric types (Value metrics)
├── plugs/             # Phoenix plugs (ThemeSelector)
├── adapter.ex         # Adapter behavior
├── field.ex           # Field behavior and config schema
├── live_resource.ex   # LiveResource macro and core logic
├── resource.ex        # Resource data manipulation
└── router.ex          # Routing helpers and macros

priv/
├── gettext/           # Translation files
├── static/            # Static assets (compiled JS, images)
└── templates/         # Default layout templates

assets/js/             # JavaScript for LiveView hooks
```

The library provides:
- **Core abstractions**: `LiveResource`, `Field`, `Filter`, `ItemAction`, `ResourceAction`, `Metric`
- **Adapter system**: Pluggable data layer support (Ecto by default, Ash via community project [ash_backpex](https://github.com/enoonan/ash_backpex))
- **UI components**: Reusable Phoenix Components for rendering admin interfaces
- **Routing helpers**: Macros for defining RESTful LiveView routes

**2. Demo Application (`/demo`)**

The demo directory contains a full Phoenix application that demonstrates Backpex capabilities:

```
demo/
├── lib/
│   ├── demo/              # Application logic (schemas, contexts)
│   │   └── helpdesk/      # Example domain (Tickets)
│   └── demo_web/          # Web layer
│       ├── components/    # App-specific components
│       ├── live/          # LiveResource implementations
│       │   ├── user_live.ex
│       │   ├── post_live.ex
│       │   ├── product_live.ex
│       │   └── ...
│       ├── filters/       # Custom filter implementations
│       ├── item_actions/  # Custom item actions
│       ├── resource_actions/  # Custom resource actions
│       └── router.ex      # Route definitions
├── priv/
│   ├── repo/              # Database migrations and seeds
│   └── static/            # Static assets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [naymspace/backpex](https://github.com/naymspace/backpex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
