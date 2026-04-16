---
trigger: always_on
description: The Multistep Product Template (`product.pdp_multistep_v1.json`) is a custom Shopify product page template that provides a guided, step-by-step customization experience for complex products with multiple options, variants, accessories, and smart features. The template transforms the standard product page into an immersive fullscreen experience with 5 distinct steps.
---

# Multistep Product Template Documentation

## Overview

The Multistep Product Template (`product.pdp_multistep_v1.json`) is a custom Shopify product page template that provides a guided, step-by-step customization experience for complex products with multiple options, variants, accessories, and smart features. The template transforms the standard product page into an immersive fullscreen experience with 5 distinct steps.

## Template Structure

### File Location
- **Template**: `/templates/product.pdp_multistep_v1.json`
- **Main Form Snippet**: `/snippets/product-multistep-form.liquid`
- **Step Snippets**: `/snippets/product-multistep-step1.liquid` through `step5.liquid`
- **JavaScript**: `/assets/product-multistep.js`
- **CSS**: `/assets/product-multistep.css`
- **Hero Section**: `/sections/section-multistep-hero.liquid`

### Template Configuration

The template consists of two main sections:

1. **Section Liquid Custom** (`section_liquid_custom`):
   - Contains critical CSS for fullscreen mode
   - Hides default product media containers
   - Manages fullscreen body state
   - Controls header/footer visibility during customization

2. **Main Product Section** (`main`):
   - Type: `product`
   - Layout: `none@columns` (no default product layout)
   - Contains single block: `multistep_form`
   - Renders `product-multistep-form` snippet

## The 5-Step Flow

### Step 1: Product Hero / Introduction
- **Purpose**: Initial product showcase and feature discovery
- **Snippet**: `product-multistep-step1.liquid`
- **Features**:
  - Product title, cutline, and price display
  - Hero image with interactive feature points (plus icons)
  - Feature list with modal popups
  - "Customize" button to start customization
- **Navigation**: Clicking "Customize" or URL parameter `?step=2` advances to Step 2
- **Fullscreen**: Not active (normal page view)
- **Dynamic Price Display**:
  - **CRITICAL**: Price is calculated in **Liquid on initial render** to prevent flash of incorrect price (FOUC)
  - Scans all **available variants only** to find the lowest price variant
  - Displays: `"{lowest_price}$ {compare_at_price}$"` (with strikethrough on compare_at if exists)
  - Strikethrough price styling: `font-weight: 400; font-size: 0.7em;` (smaller and lighter)
  - Or: `"{lowest_price}$"` if no compare_at_price
  - Liquid logic: Loops through `product.variants`, filters by `variant.available`, finds variant with lowest `variant.price`
  - JavaScript `updateStep1Price()` method exists as fallback but should not be needed since Liquid renders correctly
  - **NO VISIBLE PRICE CHANGE**: Price renders correctly from server-side, no client-side update flash

### Step 2: Color & Shell Selection
- **Purpose**: Select fabric color and shell color options
- **Snippet**: `product-multistep-step2.liquid`
- **Features**:
  - Dual swatch selection (color + shell color)
  - Dynamic image slider filtered by selections
  - Real-time variant matching and price updates
  - Image filtering based on alt text patterns (`shell_color / fabric_color`)
- **Validation**: Both color and shell color must be selected to proceed
- **Variant Selection**: Partial variant matching (color + shell, no smart option yet)
- **Fullscreen**: Active (hides header, fullscreen mode)

### Step 3: Smart Option Selection
- **Purpose**: Choose between smart and non-smart product versions
- **Snippet**: `product-multistep-step3.liquid`
- **Features**:
  - "Add Smart Capabilities" button
  - "Skip" option for non-smart version
  - **Dynamic Upgrade Price Display**: Calculates price difference between smart and classic variants for selected color combination
  - Final variant selection (color + shell + smart option)
- **Variant Selection**: Complete variant matching with all three options
- **Fullscreen**: Active
- **Dynamic Upgrade Price Calculation**:
  - Finds all variants matching the selected shell color + fabric color from step 2
  - Identifies lowest price variant (classic) and highest price variant (smart) for that color combo
  - Calculates: `smart_price - classic_price = upgrade_difference`
  - Calculates: `smart_compare_at_price - classic_price = strikethrough_difference` (if smart has compare_at)
  - Displays: `"{upgrade_difference}$ {strikethrough_difference}$"` (with strikethrough on second price if compare_at exists)
  - Strikethrough price styling: `font-weight: 400; font-size: 0.7em;` (smaller and lighter)
  - Or: `"{upgrade_difference}$"` if no compare_at_price on smart variant
  - Updates automatically when step 3 is shown and when colors change in step 2 (if already on step 3)
  - Method: `updateStep3UpgradePrice()`

### Step 4: Accessories Selection
- **Purpose**: Browse and add optional accessories
- **Snippet**: `product-multistep-step4.liquid`
- **Features**:
  - Accessory collection display
  - Checkbox selection with auto-uncheck after 2 seconds
  - Variant options for accessories (if applicable)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bareta006) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
