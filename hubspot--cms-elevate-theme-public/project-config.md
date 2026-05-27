---
trigger: always_on
description: This rule guides implementing CSS Grid layouts using template-embedded conditionals within HubSpot CMS themes, implementing RFC 0942 Grid Layouts properly.
---

# HubSpot Grid Layout Template Implementation Guide

You are assisting with implementing **CSS Grid layout sections within templates** for HubSpot CMS themes. This implements RFC 0942 Grid Layouts by embedding grid-specific code directly within template files using **conditional logic**, while preserving existing Bootstrap 2 implementations.

Here is the doc link for RFC 0942 Grid Layouts: https://product.hubteam.com/docs/content/rfcs/0942-grid-layouts.html. Use the Hubspot MCP server to read this documentation before starting.

## Overview & Methodology

### Implementation Goals
- Embed **CSS Grid layout code directly within templates** using conditional logic
- Use **template-level conditionals** to choose between legacy and grid implementations
- Maintain **complete separation** between Bootstrap 2 and Grid Layout approaches within same template
- Enable modern CSS Grid layouts without disrupting existing functionality
- Keep all grid and legacy code within the same template file for easier maintenance

### Template-Embedded Grid Approach
1. **Modify template files directly** - Add grid conditionals within existing template structure
2. **Use `{% if grids %}` conditionals** - Separate grid and Bootstrap implementations completely
3. **Embed complete grid sections** - Write full grid section code within conditional blocks
4. **Preserve Bootstrap references** - Keep existing Bootstrap sections as `include_dnd_partial` calls
5. **Maintain single template files** - All layout variations contained within same template

## RFC 0942 Implementation Context

### Beta Implementation Phase
This implementation follows **RFC 0942 Grid Layouts** adoption strategy:
- Currently in **beta phase** with gated rollout starting in Elevate theme
- Uses temporary `{% if grids %}` parameter for conditional rendering
- Post-beta: Parameter will be removed and replaced with automatic content tree detection
- Emergency rollback capability required during beta phase

### Migration Strategy
- Grid implementations are **additive** during beta (no breaking changes)
- Future migration phase will handle existing Bootstrap 2 content conversion
- Template retrofitting required post-alpha for existing implementations
- See issue #2158 for comprehensive deprecation planning

## Template Structure Strategy

### Template File Focus
Modify existing template files to include embedded grid conditionals:

```
src/unified-theme/templates/
├── contact.hubl.html          # Contains both grid and Bootstrap implementations
├── features.hubl.html         # Contains both grid and Bootstrap implementations
├── pricing.hubl.html          # Contains both grid and Bootstrap implementations
└── home.hubl.html             # Contains both grid and Bootstrap implementations
```

## Template Implementation Patterns

### Simple Single-Section Template
Templates use a SINGLE `{% if grids %}` conditional wrapping complete `dnd_area` blocks:

```hubl
<!-- templates/contact.hubl.html -->
{% extends "./layouts/base.hubl.html" %}

{% block body %}
  {% if grids %}
    {# Grid Implementation - Complete dnd_area with grid sections #}
    {% dnd_area "dnd_area" 
      label="Main content" 
    %}
      {% dnd_grid_section
        content_width={{ section_width_narrow }},
        background_layers=[
          {
            "type": "color",
            "value": light_section_3_background_color
          }
        ]
      %}
        {% dnd_grid_container
           type="vertical_stack",
           gap={{ spacing_0 }}
        %}
          {% dnd_grid_module
            path="../components/modules/Anchor",
            anchor={{ scaffold_content.anchor.anchor_id }}
          %}
          {% end_dnd_grid_module %}
          {% dnd_grid_module
            path="../components/modules/Heading",
            headingAndTextHeadingLevel="h1",
            headingAndTextHeading={{ template_translations.contact_heading.message }}
          %}
          {% end_dnd_grid_module %}
        {% end_dnd_grid_container %}
      {% end_dnd_grid_section %}
    {% end_dnd_area %}
  {% else %}
    {# Bootstrap Implementation - Complete dnd_area with Bootstrap sections #}
    {% dnd_area "dnd_area" 
      label="Main content" 
    %}
      {% include_dnd_partial name="contact" %}
    {% end_dnd_area %}
  {% endif %}
{% endblock %}
```

### Multi-Section Template Pattern
For templates with multiple sections, use a SINGLE `{% if grids %}` conditional wrapping complete `dnd_area` blocks:

```hubl
<!-- templates/home.hubl.html with multiple sections -->
{% extends "./layouts/base.hubl.html" %}

{% block body %}
  {% if grids %}
    {# Grid Implementation - Complete dnd_area with all grid sections #}
    {% dnd_area "dnd_area" 
      label="Main content" 
    %}
      {% dnd_grid_section %}
        {% dnd_grid_container
           type="grid",
           rows=["1fr"],
           columns=["1fr", "1fr"] 
        %}
          {% dnd_grid_module path="../modules/hero-content" %}
          {% end_dnd_grid_module %}
          {% dnd_grid_module path="../modules/hero-image" %}
          {% end_dnd_grid_module %}
        {% end_dnd_grid_container %}
      {% end_dnd_grid_section %}

      {% dnd_grid_section %}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HubSpot/cms-elevate-theme-public](https://github.com/HubSpot/cms-elevate-theme-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
