---
trigger: always_on
description: Scaffold a new section
---

# Scaffold a Section

When scaffolding a new section, follow these guidelines to create a consistent, well-structured foundation. Focus on creating the basic structure and necessary content without worrying about translations, scaffolding, or context values.

## Key Principles
- Create minimal, well-organized file structure
- Add TODOs and comments for future implementation
- Focus on building out the section using DND tags with hardcoded content
- Set up proper section template annotations
- Never hardcode colors or unnecessary styles
- Use existing theme modules over default modules
- Always check the codebase for existing modules before assuming default modules exist
- Always check existing sections to see what fields they're passing into modules

## Core Concepts

### Template Annotations
At the top of a section is the file's [template annotations](mdc:https:/developers.hubspot.com/docs/guides/cms/content/templates/drag-and-drop/sections#section-template-files) which includes meta information about the section that is then used to display the section in the page editor when a user is editing a page. Below is an example of a template annotation for a section.

```
<!--
  templateType: section
  label: Name of the section
  isAvailableForNewContent: true
  screenshotPath: ../images/section-previews/screenshot-of-the-section.png
  description: "Description of the section"
-->
```

### DND Area Tags
A section is made up of [drag and drop HubL tags](mdc:https:/developers.hubspot.com/docs/reference/cms/hubl/tags/dnd-areas). A `dnd_section` is made up of `dnd_row`s, `dnd_column`s, and `dnd_module`s. You can read more about the HubL tags, their available parameters, and their nesting rules [here](mdc:https:/developers.hubspot.com/docs/reference/cms/hubl/tags/dnd-areas). Below is a very basic example of a section.

```
{% dnd_section %}
  {% dnd_column %}
    {% dnd_row %}
      {% dnd_column %}
        {% dnd_module
          path="../path/to/module.html"
          fieldName="fieldValue"
        %}
        {% end_dnd_module %}
      {% end_dnd_column %}
    {% end_dnd_row %}
  {% end_dnd_column %}
{% end_dnd_section %}
```

### Modules Available to Be Used in a Section
A section can be built with theme and/or default modules.

#### Theme Modules (Preferred)
A theme module is a module that exists within a website theme and can only be used in that particular website theme. These files live in the same folder as the theme. You can learn more about theme modules [here](mdc:https:/developers.hubspot.com/docs/guides/cms/content/themes/overview).

Theme modules are located inside the components/modules directory and should be referenced with the path `../components/modules/ModuleName`. These modules:
- Are specific to the website theme
- Should be used before considering default modules
- May be new and not used in other sections yet - check the components/modules directory
- Are referenced with `../` in the path


#### Default Modules
Default modules are modules that can be used in any themes. These files live separately from themes. You can learn more about default modules and what their different fields are [here](mdc:https:/developers.hubspot.com/docs/reference/cms/modules/default-modules).

Default modules (prefixed with @hubspot/) should only be used when a suitable theme module doesn't exist. Always check the theme modules directory first.

When using default HubSpot modules, make sure to use the exact module names as they appear in existing code:

- For images, use `@hubspot/linked_image` (NOT `@hubspot/image` because that doesn't exist)
- For image galleries, use `@hubspot/image_grid`
- For forms, use `@hubspot/form`

Always check existing sections first to see how modules are implemented before adding new ones. Grep for "@hubspot/" to find examples of default modules in use.


### How to Reference Module Fields From a Section
Each module is made up of fields. Fields are what are used by a user to edit the module's contents. You can learn more about the fields available for HubL modules [here](mdc:https:/developers.hubspot.com/docs/reference/cms/fields/module-theme-fields) and the fields available for React modules [here](https://developers.hubspot.com/docs/guides/cms/react/modules#module-fields). When adding a module to a section, you can reference the module's fields to change the module field values for that particular section. Below is an example of a module's fields.json for a HubL module, equivalent fields.tsx file for a React module, and how that module is referenced in the section.

**Fields.json:**
```
[
  {
    "label": "Icon",
    "name": "icon",
    "type": "icon",
    "icon_set": "fontawesome-6.4.2",
    "default": {
      "name": "arrow-alt-circle-up",
      "type": "SOLID",
      "unicode": "f35b"
    }
  }
]
```

**Fields.tsx:**
```
import {
  IconField,
} from '@hubspot/cms-components/fields';
export const fields = (
  <ModuleFields>
    <IconField
      label='Icon'
      name='icon'
      iconSet='fontawesome-6.4.2'
      default={{
        name: 'arrow-alt-circle-up',
        type: 'SOLID',
        unicode: 'f35b'
      }}
    />
  </ModuleFields>
);
```

**Reference in section:**
```
{% dnd_section %}
  {% dnd_column %}
    {% dnd_row %}
      {% dnd_column %}
        {% dnd_module

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HubSpot/cms-elevate-theme-public](https://github.com/HubSpot/cms-elevate-theme-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
