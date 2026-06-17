---
trigger: always_on
description: Guidance for Claude Code when working with this Rails + InertiaJS React/TypeScript codebase.
---

# CLAUDE.md

Guidance for Claude Code when working with this Rails + InertiaJS React/TypeScript codebase.

## Core Rules

- Always include a confidence interval for how confident you are that the proposed solution is optimal
- DO NOT run `git add` or `git commit` without my approval
- DO NOT over-engineer. Simple solutions > complex abstractions
- Be critical — challenge suggestions that lead to poor code quality or architecture
- Before implementing, ask clarifying questions about assumptions
- Consult context7 skill for related topics before coding. If unavailable, do web research. Don't add anything you don't know
- Read `docs/tailwind.md` before writing frontend code. Read `docs/STYLE.md` before writing Ruby
- Read `docs/auth-tenant-guide.md` before adding models, controllers, or features that involve tenant data, authentication, or authorization. It covers the Identity/User/Account pattern, scoping rules, and has a safety checklist
- Read `docs/analytics.md` before modifying analytics tracking, the dashboard, visit/event models, or the Ahoy store. It covers the dedicated database, query pipeline, client-side tracker, live view, and how to extend safely
- Self-critique your work when finishing a task
- Simplicity criterion: All else being equal, simpler is better. A small improvement that adds ugly complexity is not worth it. Conversely, removing something and getting equal or better results is a great outcome — that's a simplification win. When evaluating whether to keep a change, weigh the complexity cost against the improvement magnitude. A improvement that adds 20 lines of hacky code? Probably not worth it. A improvement from deleting code? Definitely keep. An improvement of ~0 but much simpler code? Keep.
- **DO NOT GUESS** — If you can't test or verify something after 2 attempts, ask for help with specific instructions

## Development Workflow

### Before Coding
- Research: Check existing patterns first, then docs
- Questions: Clarify before implementing assumptions

### Backend
- Follow DHH/Rails conventions strictly (see patterns below)
- Use `params.expect` (Rails 8+), not `params.require().permit()`
- Inertia props: use **snake_case** in Ruby — `prop_transformer` auto-converts to camelCase
- When implementing HTTP/CDN caching for Inertia routes, treat full HTML document requests and `X-Inertia` requests as different response types. Only cache explicitly public anonymous HTML responses; do not apply public/shared caching to Inertia requests or to HTML that can vary based on authenticated header/nav state (`Current.identity`, flash, cookies, session). In this app, use `Current.identity.present?` as the main personalization/auth boundary for cache safety on public pages
- Never run migrations automatically — ask first
- Always use gem's official generators — never manually write files that generators create
- Generate migrations with Rails generators like `bin/rails generate migration ...` or `bin/rails generate model ...`; do not create migration files by hand
- Run `bin/rubocop --autocorrect` after changes

### Frontend
- Follow Next.js naming: files in `kebab-case.tsx`, components in `PascalCase`, props in `camelCase`
- Never modify `app/frontend/components/ui/` (shadcn managed)
- Run `npm run check && npm run lint && npm run format:check` after changes — Prettier failures are common

### Strong Parameters

```ruby
# Rails 8+ style
def item_params
  params.expect(item: [:name, :email, :phone, tags: []])
end

# Nested attributes use double brackets [[ ]]
def parent_params
  params.expect(parent: [:name, children_attributes: [[:id, :value, :_destroy]]])
end
```

Frontend must wrap form data:
```tsx
const { data, post, transform } = useForm({ name: "", email: "" })
transform((data) => ({ item: data }))
```

### Inertia Props Convention

```ruby
# Controller (snake_case) — auto-transforms to camelCase for JS
render inertia: "items/show", props: {
  item: item_props(item),
  related_items: related.map { |i| item_props(i) },
  total_count: items.count
}
```

```tsx
// Frontend receives camelCase
interface Props {
  item: Item
  relatedItems: Item[]    // from related_items
  totalCount: number      // from total_count
}
```

## Rails Patterns

### REST Actions Only (DHH Philosophy)

Only use the 7 REST actions: `index`, `show`, `new`, `edit`, `create`, `update`, `destroy`. When you need a custom action, make a new controller.

```ruby
# WRONG — custom actions
class ItemsController
  def archive; end
  def publish; end
end

# CORRECT — separate controllers with REST actions
class Items::ArchivesController
  def create; end      # POST /items/:item_id/archives
  def destroy; end     # DELETE (unarchive)
end

class Items::PublicationsController
  def create; end      # POST /items/:item_id/publications
end
```

Think in REST nouns, not verbs. Controller proliferation is good.

### Rich Domain Models Over Service Objects

```ruby
# CORRECT — rich model
@item.archive
@item.compute_score

# WRONG — unnecessary service layer
ItemArchiveService.new(@item).call
```

Extract objects only when complexity demands it. Use concerns for shared model behavior.

## Project Structure

```
app/frontend/
├── components/
│   └── ui/              # shadcn/ui (NEVER MODIFY)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darkamenosa/contextqmd-registry](https://github.com/darkamenosa/contextqmd-registry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
