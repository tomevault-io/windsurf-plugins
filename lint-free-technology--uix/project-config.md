---
trigger: always_on
description: UI eXtension (UIX) is a custom integration for [Home Assistant](https://www.home-assistant.io/) that enables advanced CSS customisation across the entire Home Assistant UI. It comes from the heritage of [card-mod](https://github.com/thomasloven/lovelace-card-mod) by [@thomasloven](https://github.com/thomasloven) and extends it with new features such as Jinja2 macros, improved DOM navigation, and in-browser debugging helpers.
---

# UIX Agent Guide

## What is UI eXtension (UIX)?

UI eXtension (UIX) is a custom integration for [Home Assistant](https://www.home-assistant.io/) that enables advanced CSS customisation across the entire Home Assistant UI. It comes from the heritage of [card-mod](https://github.com/thomasloven/lovelace-card-mod) by [@thomasloven](https://github.com/thomasloven) and extends it with new features such as Jinja2 macros, improved DOM navigation, and in-browser debugging helpers.

- **Full documentation:** <https://uix.lf.technology/>
- **UIX Guides** (community-curated examples and tutorials): <https://uix-guides.lf.technology>
- **FAQ:** <https://uix.lf.technology/faq>
- **Quick Start:** <https://uix.lf.technology/quick-start>
- **GitHub repository:** <https://github.com/Lint-Free-Technology/uix>
- **GitHub discussions:** <https://github.com/Lint-Free-Technology/uix/discussions>

---

## Installation

### Via HACS (recommended)

Add `https://github.com/Lint-Free-Technology/uix` as a custom HACS repository (type: `Integration`), then download it and add the **UI eXtension** service in Home Assistant **Settings → Devices & Services**.

### Manual

Copy the contents of [`custom_components/uix`](https://github.com/Lint-Free-Technology/uix/tree/master/custom_components/uix) into `<config>/custom_components/uix/`, restart Home Assistant, then add the service as above.

---

## How Users Apply UIX

UIX is configured through a `uix:` key added to a card, entity, badge, or element in a Lovelace/dashboard YAML configuration. It requires no resource URL management—UIX handles that automatically.

### Basic card style

```yaml
type: entities
show_header_toggle: false
entities:
  - light.bed_light
uix:
  style: |
    ha-card {
      background: red;
    }
```

### Using CSS variables

Home Assistant themes expose CSS variables that UIX can both read and override:

```yaml
uix:
  style: |
    ha-card {
      --ha-card-background: teal;
      color: var(--primary-color);
    }
```

### Styling individual entities

In `entities` and `glance` cards each entity row can be styled independently. Styles are injected into a shadow root so the bottommost element is `:host`:

```yaml
type: entities
entities:
  - entity: light.bed_light
    uix:
      style: |
        :host { color: red; }
  - entity: light.ceiling_lights
    uix:
      style: |
        :host { color: green; }
```

This also applies to view badges and elements in `picture-elements` cards.

---

## DOM Navigation and Shadow Roots

Home Assistant makes heavy use of the [shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM). To style elements inside a shadow root, make `style:` a **dictionary** instead of a string.

- Each **key** is a selector that navigates down through the DOM.
- A dollar sign `$` in the key replaces a `#shadow-root` crossing.
- A key of `.` (period) selects the current root element.
- Selector steps are separated by spaces; only the **first** match is followed at each intermediate step, but the **final** step matches **all** elements.
- A key may begin with `&` to filter the initial element before any traversal (see the [`&` host/element filter](#-hostelement-filter) section below).

### Example: styling `<h3>` inside a markdown card

```yaml
type: markdown
content: |-
  # Example
  ### This heading will be purple
uix:
  style:
    "ha-markdown $": |
      h3 {
        color: purple;
      }
    ".": |
      ha-card {
        background: teal;
      }
```

### Chaining and load-order stability

Breaking a long chain into several dictionary levels lets UIX retry each step independently, which is more reliable when elements load asynchronously:

```yaml
# Stable: UIX can retry from ha-map $ if ha-entity-marker hasn't loaded yet
uix:
  style:
    "ha-map $":
      "ha-entity-marker $":
        div: |
          color: red;
```

### `&` host/element filter

A path key may begin with `&` as its first step to filter the initial element before any traversal. If the initial context is a shadow root, the `&` filter is tested against the **host** element; if it is a regular element, it is tested against that element.

Supported tokens (all present tokens must match):

- `tagname` — element local name match
- `.classname` — class list check
- `#id` — element ID match
- `[attr]`, `[attr=val]`, `[attr^=val]`, `[attr$=val]`, `[attr*=val]`, `[attr~=val]`, `[attr|=val]` — attribute checks

Tokens may be combined (e.g. `&ha-dialog.my-class[data-type="video"]`). Selectors containing spaces are **not** supported. Class selectors may optionally be wrapped in parentheses: `&(.my-class)` equals `&.my-class`.

This is primarily useful in themes to scope a style path to a specific host class or attribute:

```yaml
# Style a dialog only when it has the class type-hui-dialog-web-browser-play-media
my-awesome-theme:
  uix-theme: my-awesome-theme
  uix-dialog-yaml: |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lint-Free-Technology/uix](https://github.com/Lint-Free-Technology/uix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
