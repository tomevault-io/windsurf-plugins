---
trigger: always_on
description: Intent reference: navigation patterns and trade-offs, taxonomy design, mental model theory, wayfinding principles, search behavior models. Load when designing navigation, site structure, taxonomy, or information hierarchy.
---


# Information Architecture

## Navigation Patterns

Every navigation pattern is a trade-off between findability, scalability, and cognitive load. There is no universally correct pattern — there's the right pattern for your content, users, and context.

### Hierarchical (Tree)

The most common pattern. Content is organized in nested categories: top-level → subcategory → item.

**When it works:** Large content sets with clear categorical relationships. Users have a general sense of what category their item belongs to. Content creators can maintain consistent categorization.

**When it fails:** When categories overlap significantly (is a "wireless mouse" in Accessories, Computers, or Peripherals?). When the hierarchy is deeper than 3-4 levels — users lose orientation quickly. When the hierarchy reflects organizational structure rather than user mental models ("Products" → "Business Unit A" → "Division 2" → "Team Alpha's Output").

**Watch for:** The "miscellaneous" drawer. If you have a category called "Other" or "General" that keeps growing, your hierarchy isn't working. Also: category names that mean something to the organization but nothing to users.

### Hub-and-Spoke

A central hub connects to independent sections. Each section is largely self-contained. Users return to the hub to navigate between sections.

**When it works:** Mobile apps with distinct functional areas (Messages, Camera, Profile). Products where tasks are independent — you don't need to combine search results with your shopping cart. Kiosk interfaces and set-top boxes where the input model favors focused navigation.

**When it fails:** When users need to move fluidly between sections. When tasks span multiple sections. When the "hub" becomes a dumping ground for everything that doesn't fit in a spoke.

**Watch for:** The desire to add cross-links between spokes. Once spokes start linking to each other extensively, the hub-and-spoke model is fighting the user's actual workflow. Consider switching to a different pattern.

### Flat

All content is at the same level. No hierarchy. Often paired with powerful search/filter.

**When it works:** Homogeneous content sets (a photo gallery, a list of transactions, a feed of posts). Content that doesn't have natural categories. Products with excellent search infrastructure.

**When it fails:** Diverse content types. Large content sets without strong search/filter. Users who browse rather than search.

**Watch for:** The illusion of flatness. Many "flat" architectures are actually filtered hierarchies — the user selects filters that create ad-hoc categories. That's fine, but the filter design is now your navigation design.

### Faceted

Multiple independent dimensions for filtering the same content set. Users combine facets freely (color + size + price + brand).

**When it works:** E-commerce, search results, large databases with multiple attributes. When different users want to slice the same content differently. When the content has natural, independent attributes.

**When it fails:** When facets aren't independent (selecting "red" and "small" leaves zero results because small items don't come in red). When there are too many facets — 15 filter dimensions overwhelm rather than help. When the vocabulary is inconsistent across facets.

**Watch for:** Empty states. Faceted navigation creates combinatorial explosion — many facet combinations will return zero results. Design for graceful degradation: show result counts per facet before selection, disable facets that would return zero.

### Dashboard

Multiple content types displayed simultaneously, typically with summary views that link to detail.

**When it works:** Monitoring and analytics products. Executive overviews. Products where users need to scan multiple information streams quickly. Return-visit products where users want a status snapshot.

**When it fails:** When the dashboard becomes the entire product — dashboards that require scrolling through 15 widgets have become a flat architecture by accident. When every stakeholder demands their metric on the dashboard, resulting in information overload. When the dashboard shows data but doesn't enable action.

**Watch for:** Dashboard-driven design, where every new feature gets a widget on the dashboard instead of its own proper location. Also: dashboards that show the same data to everyone when different roles need different views.

---

## Taxonomy Design

Taxonomy is the art of naming and grouping things so that users can find them. Get taxonomy wrong and no amount of visual design will save the navigation.

### Top-Down vs. Bottom-Up

**Top-down** starts with organizational logic: What are the major categories? How do they subdivide? This approach works when domain experts understand the structure well and users share that understanding. Risk: imposing a structure that makes sense internally but not to users.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghaida/intent](https://github.com/ghaida/intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
