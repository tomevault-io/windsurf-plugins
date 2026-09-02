---
trigger: always_on
description: This file is the canonical, self-contained guidance for the PostnHost mountable Rails engine. Paths in this document are relative to the engine root.
---

# AGENTS.md

This file is the canonical, self-contained guidance for the PostnHost mountable Rails engine. Paths in this document are relative to the engine root.

## Repository Boundary

The engine owns reusable CMS models, controllers, services, generators, views, assets, translations, and engine tests. Host applications own deployment, database operations, backup infrastructure, monitoring mounts, branding overrides, and production credentials.

Never rely on host-application files, parent instructions, or parent-relative runtime paths. The self-hosted application lives in the separate `postnhost-app` repository, and this engine must remain independently buildable and testable.

## Project Overview

PostnHost is a mountable Rails engine with Hotwire, Stimulus, Tailwind CSS, PaperTrail, CarrierWave, and optional OpenAI-backed translation workflows.

- Supported Ruby: 3.4 and 4.0.
- Supported Rails: 7.2, 8.0, and 8.1.
- Namespace: `Postnhost`.
- Authentication: custom session-based authentication with `has_secure_password`.
- Authorization: authenticated CMS/internal flows use `before_action :authenticate_user!`.
- Assets: precompiled Tailwind CSS and a bundled JavaScript ES module ship in the gem.
- Tests: RSpec model, service, concern, helper, request, generator, and Selenium system specs.
- Compatibility: Appraisal gemfiles exercise the supported Rails matrix against the dummy application.

## Working Rules

### Read Before Editing

- Read every relevant file in full before changing it.
- Search for implementations, references, tests, generators, and copied-view manifests before adding code.
- Inspect `git status` and preserve unrelated user changes.
- Consider multiple approaches and prefer the smallest cohesive design.
- Review the final diff and remove code, comments, assets, or compatibility branches that do not serve the requested behavior.

### File Hygiene

- New files must use mode `644` unless intentionally executable.
- Never commit dummy uploads, databases, logs, coverage, temporary output, editor files, or JavaScript source maps.
- Generated distributable CSS and JavaScript under `app/assets/builds/postnhost/` are intentional gem inputs and must be committed after source changes.
- Keep fixtures under `spec/fixtures/`, not under dummy runtime directories.

### Quality Checks

- Run `bundle exec rubocop -a` before finishing Ruby changes.
- Run focused specs while iterating, then the relevant complete engine suite.
- Rebuild packaged JS and CSS when their inputs change.
- Build and inspect the gem when packaging, assets, metadata, generators, or migrations change.

## Architecture

### Models

- Persistent domain rules belong in models: validations, associations, callbacks, scopes, and simple record behavior.
- Keep models focused and extract behavior only when it is genuinely shared.
- Use `touch: true` to represent cache invalidation dependencies.
- Do not wrap Active Record relations with `Array(...)`; handle `nil` explicitly and preserve relation-friendly APIs.

### Services

- Workflow orchestration, external integrations, multi-model commands, and procedural operations belong under `app/services/postnhost/`.
- Inherit application services from `Postnhost::BaseService`.
- Expose `.call(...)` through `BaseService.call`, with one public instance `call` method.
- Return `Postnhost::ServiceResult` when callers need `value`, `errors`, and `status`.
- Keep initializers explicit for required inputs and injected dependencies.
- Do not call `super()` unless a parent initializer actually has state to initialize.
- Split long step-based workflows into descriptive classes under a `steps/` directory.

### Concerns

- Use `ActiveSupport::Concern` for cohesive behavior intentionally mixed into multiple models or controllers.
- Keep one responsibility per concern and namespace it under `Postnhost` where appropriate.
- Do not use concerns as command or workflow buckets.

### Controllers

- Keep controllers under 100 lines and delegate domain behavior to models or services.
- Use strong parameters and explicit `before_action` loaders.
- Protect internal controllers with `before_action :authenticate_user!`.
- Prefer redirects over renders after state changes.
- Prefer standard Rails responses and Turbo Streams over client-managed API flows.

## Views and Generators

### ERB

- Never assign variables in ERB templates.
- Keep templates declarative; prepare data in controllers, models, helpers, or presenters.
- Use `form_with`, `button_to`, Rails helpers, partials, and flash messages. Never add raw `<form>` elements.
- Use semantic HTML and add `cursor-pointer` to clickable controls.
- Public helpers must be limited to data, URLs, sanitization, and formatting; keep public presentation markup and Tailwind classes in templates.

### Public View Copying

- Host applications can copy public templates with `rails g postnhost:views`.
- Whenever a public template under `app/views/postnhost/public/` or public layout partial under `app/views/layouts/postnhost/` is added, moved, or removed, update `lib/generators/postnhost/views/views_generator.rb` and its specs.
- Keep minimal and full view scopes internally consistent.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [postnhost/postnhost](https://github.com/postnhost/postnhost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
