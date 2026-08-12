---
trigger: always_on
description: Baymard-grade e-commerce UX standards for Best Bottles — filtering, product lists, PDP, search, navigation, cart, accounts, mobile, and B2B wholesale patterns. Sourced from 200,000+ hours of usability testing across 327 benchmarked sites.
---


# E-Commerce UX Gold Standard (Baymard Institute)

These standards govern all catalog, product, search, navigation, and checkout UI work on Best Bottles. They are distilled from Baymard Institute research (200,000+ hours of usability testing, 327 top-grossing sites benchmarked, 700+ UX guidelines). Violations cause measurable abandonment — treat these as hard requirements, not suggestions.

Sources: [baymard.com/blog](https://baymard.com/blog), [Product List](https://baymard.com/blog/collections/product-list), [Product Page](https://baymard.com/blog/collections/product-page), [Cart & Checkout](https://baymard.com/blog/collections/cart-and-checkout), [Homepage & Category](https://baymard.com/blog/collections/homepage-and-category), [Accounts & Self-Service](https://baymard.com/blog/collections/accounts-and-self-service), [Popular](https://baymard.com/blog/popular), [B2B Research](https://baymard.com/research/business-to-business)

---

## 1. Faceted Filtering & Product Lists

### Filter Logic
- **One facet = one product attribute.** Never inflate a facet by pulling in products from other attribute dimensions (e.g., Atomizer family must only show family=Atomizer products).
- **OR logic within a group, AND logic across groups.** 15% of sites incorrectly force mutually exclusive selection within filter groups — always use multi-select checkboxes.
- **Show live product counts** next to every option — `Blue (47)`. Update dynamically. Hide zero-count options entirely.
- **Display applied filters in an overview chip bar** (32% don't use best practices for this). Each chip must have a one-click remove action.

### Filter Layout
- **Order facets by usage frequency.** Most-used (price, applicator type, capacity) at top. Re-evaluate quarterly.
- **Promote key filters** as horizontal pill toggles above the grid (All Bottles | Roll-On | Spray). 61% of sites fail to promote important filters.
- **5 essential filter types** must be present for product listings. 57% of sites don't offer all five. Ensure the core filter types users expect are available.
- **Filters for all displayed list-item info.** If capacity, neck thread, color appear on the card, matching filters must exist. 38% of sites don't align filters with displayed attributes.
- **Always explain industry-specific filters.** 62% don't. For Best Bottles: explain what "neck thread size" means with a tooltip or inline hint.
- **Desktop: persistent sidebar.** Never hide filters behind a "Filter" button on desktop — the space is available.
- **Mobile: full-screen filter modal.** Do not use inline accordions that push content down. Collapse to a bottom sheet or modal.
- **Avoid horizontal filtering toolbars** as sole filter mechanism — they hide depth and truncate on smaller viewports.

### Product List Items
- **Combine variations into one list item.** Show one card per product group (e.g. "Cylinder 9ml Clear – 6 variants"), not separate cards per SKU variant. 12% of sites don't — this clutters lists and hurts product finding.
- **5 required attributes per card:** thumbnail, product title, price, variant count, one differentiating spec (capacity/color/applicator for Best Bottles).
- **Display "Price Per Unit"** for multiquantity items. When showing case pricing, always also show per-unit price. 86% of sites don't.
- **Consistent attribute display** across every card in a grid. Never show specs on some cards but not others. (64% get at least one of the two key list-item design principles wrong.)
- **At least 3 product thumbnails** visible per list item for visually-driven products. Let users preview alternate images without opening the PDP.
- **Color swatches** in list items for visually-driven products — 57% of mobile sites don't show all swatches.
- **Highlight items already in the user's cart** with a subtle indicator on the list card. 96% of sites don't — this reduces duplicate adds and improves orientation.
- **Avoid Quick Views for spec-driven products.** Best Bottles is spec-driven (neck thread, capacity, material). Send users to the full PDP; don't use Quick View modals. 21% of spec-driven sites still use Quick Views inappropriately.
- **Synchronized hover effects and unified hit-areas.** The entire card should be clickable with a consistent hover state. 76% of sites don't synchronize hover effects across the card.
- **Scannability over density.** Bold price, muted secondary text, clear thumbnail. Users scan, they don't read line-by-line.

### Sorting & Pagination
- **Sort options:** relevance (default for search), price low→high, price high→low, newest, best-selling, name A→Z. 64% of sites don't offer all four essential sorts (price, rating, best-selling, newest).
- **Relevance sort must be diversity-based** — mix product families and types in results rather than clustering the same family together. 24% of sites don't diversify relevance sort.
- **"Load More" button** or pagination — never auto-load infinite scroll below the initial viewport. 52% of sites get default product count wrong.
- **Show total result count** prominently at top of results.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asalastudio/best-bottles-website](https://github.com/asalastudio/best-bottles-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
