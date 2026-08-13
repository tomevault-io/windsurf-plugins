---
trigger: always_on
description: This document contains coding rules and conventions for the WebWash daisyUI Starter theme that AI agents should follow when making changes.
---

# WebWash daisyUI Starter Theme - Agent Rules

This document contains coding rules and conventions for the WebWash daisyUI Starter theme that AI agents should follow when making changes.

## CVA (Class Variant Authority) Usage

### Conditionals Must Use CVA

**Rule**: Never use inline conditionals within HTML tag attributes. Always use CVA (Class Variant Authority) to handle conditional classes.

**❌ Bad:**

```twig
<h3 class="font-semibold{% if badges is empty %} relative{% endif %}">
  <a href="{{ url }}"{% if badges is empty %} class="after:absolute after:inset-0 after:content-['']"{% endif %}>
```

**✅ Good:**

```twig
{% set heading_variants =
  html_cva(
    base: 'font-semibold',
    variants: {
      hasBadges: {
        yes: '',
        no: 'relative'
      }
    }
  )
%}

<h3
  class="{{
  heading_variants.apply({
    hasBadges: has_badges ? 'yes' : 'no'
  })
  }}"
>
  <a
    href="{{ url }}"
    class="{{
    link_variants.apply({
      hasBadges: has_badges ? 'yes' : 'no'
    })
    }}"
  >

  </a>
</h3>
```

### CVA Variant Keys Use Yes/No Strings

**Rule**: CVA variant keys should use `yes`/`no` string values, not boolean values (`true`/`false`) or quoted strings (`'true'`/`'false'`).

**❌ Bad:**

```twig
clickable: { true: 'group cursor-pointer', false: '' } clickable: { 'true': 'group cursor-pointer', 'false': '' }
```

**✅ Good:**

```twig
clickable: { yes: 'group cursor-pointer', no: '' }
```

**Note**: When setting variables for CVA, convert boolean conditions to `'yes'`/`'no'` strings:

```twig
{% set is_clickable = url is not empty and url != 'No URL' ? 'yes' : 'no' %}
{% set has_badges = badges is not empty %}
{% set badge_variant = has_badges ? 'yes' : 'no' %}
```

### CVA Formatting

**Rule**: CVA definitions should use multi-line format for better readability.

**✅ Good:**

```twig
{% set card =
  html_cva(
    base: 'card flex flex-col',
    variants: {
      orientation: {
        stacked: '',
        landscape: 'md:flex-row'
      }
    }
  )
%}
```

**Rule**: CVA `.apply()` calls should use multi-line format when they contain multiple parameters or are long.

**✅ Good:**

```twig
<div
  class="{{
  card.apply(
    {
      orientation: card_orientation,
      style: card_style,
      clickable: is_clickable
    },
    card_classes|default('')
  )
  }}"
></div>
```

### Use Arrays for Long Class Strings

**Rule**: When class strings become too long (typically over 80-100 characters), use arrays instead to improve readability. Each class should be on its own line in the array.

**❌ Bad:**

```twig
variant: { primary: 'border-[var(--hgc-btn-border)] bg-[var(--hgc-btn-bg)] text-[var(--hgc-btn-label)] hover:border-[var(--hgc-btn-border-hover)]
hover:bg-[var(--hgc-btn-bg-hover)] hover:text-[var(--hgc-btn-label-hover)] focus:border-[var(--hgc-btn-border-hover)]
focus:bg-[var(--hgc-btn-bg-hover)] focus:text-[var(--hgc-btn-label-hover)] disabled:cursor-default disabled:border-[var(--hgc-btn-border-disabled)]
disabled:bg-[var(--hgc-btn-bg-disabled)] disabled:text-[var(--hgc-btn-label-disabled)]' }
```

**✅ Good:**

```twig
variant: { primary: [ 'border-[var(--hgc-btn-border)]', 'bg-[var(--hgc-btn-bg)]', 'text-[var(--hgc-btn-label)]',
'hover:border-[var(--hgc-btn-border-hover)]', 'hover:bg-[var(--hgc-btn-bg-hover)]', 'hover:text-[var(--hgc-btn-label-hover)]',
'focus:border-[var(--hgc-btn-border-hover)]', 'focus:bg-[var(--hgc-btn-bg-hover)]', 'focus:text-[var(--hgc-btn-label-hover)]',
'disabled:cursor-default', 'disabled:border-[var(--hgc-btn-border-disabled)]', 'disabled:bg-[var(--hgc-btn-bg-disabled)]',
'disabled:text-[var(--hgc-btn-label-disabled)]' ] }
```

**Note**: Arrays can be used directly in CVA variant definitions without needing to join them. CVA will handle the array format automatically. This makes long class strings much more readable and easier to maintain.

### Normalize Array/String Inputs for CVA Apply

**Rule**: When passing additional classes to `Cva::apply()` as the second argument, always normalize the input to handle both array and string formats. `Cva::apply()` expects a string (or null) for the second argument, but components may receive arrays from parent templates.

**❌ Bad:**

```twig
{% set btn_classes = btn_classes|default('') %}
<div
  class="{{
  button.apply(
    {
      size: button_size,
      variant: button_variant
    },
    btn_classes
  )
  }}"
></div>
```

**Note**: This will fail with a runtime error if `btn_classes` is passed as an array (e.g., `btn_classes: ['before:absolute', 'before:inset-0']`).

**✅ Good:**

```twig
{# Normalize btn_classes - handle both array and string formats #}
{% set additional_classes = btn_classes|default('') %}
{% if additional_classes is iterable %}
  {% set additional_classes = additional_classes|join(' ') %}
{% endif %}

<div
  class="{{
  button.apply(
    {
      size: button_size,
      variant: button_variant
    },
    additional_classes
  )
  }}"
></div>
```

**Note**: This pattern ensures the component works whether additional classes are passed as an array or a string. Always normalize external inputs before passing them to `Cva::apply()`.

## HTML Tag Attributes

### No Conditionals in Class Attributes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WebWash/ww_daisyui_starter](https://github.com/WebWash/ww_daisyui_starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
