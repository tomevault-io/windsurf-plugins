---
trigger: always_on
description: Scaffold a new template
---

# Scaffold a Template

When scaffolding a new template, follow these guidelines to create a consistent, well-structured foundation. Focus on creating the basic structure and necessary content without worrying about translations, scaffolding, or context values.

## Key Principles
- Create minimal, well-organized file structure
- Add TODOs and HubL comments `{# HubL comment #}` for future implementation
- Focus on building out the template using existing sections in the theme or with DND tags with hardcoded content if a section doesn't exist for a particular part of the template
- Set up proper template annotations
- Never hardcode colors or unnecessary styles
- Use existing theme modules over default modules
- Always check the codebase for existing modules before assuming default modules exist
- Always check existing sections and/or templates to see what fields they're passing into modules

## Core Concepts

### Template Annotations
At the top of a template is the file's [template annotations](mdc:https:/developers.hubspot.com/docs/guides/cms/content/templates/types/html-hubl-templates#template-annotations) which includes meta information about the section that is then used to display the template in the content creation flow when a user goes to choose a template to start their page with. Below is an example of a template annotation for a template.

```
<!--
  templateType: page
  label: Name of the template
  isAvailableForNewContent: true
  screenshotPath: ../images/template-previews/screenshot-of-the-template.png
-->
```

### Extending Base and Global Partials
Each of our templates extends a base file which includes a shared template structure. Within our base file there are three different blocks which allow us to optionally change the default content of those blocks for each template.

#### For page templates we should start our template with the following code:
```
<!--
  templateType: page
  label: Name of the template
  isAvailableForNewContent: true
  screenshotPath: ../images/template-previews/screenshot-of-the-template.png
-->
{% extends "./layouts/base.hubl.html" %}

{% block body %}
{% endblock body %}
```

#### For landing page templates we should start our template with the following code:
```
<!--
  templateType: page
  label: Name of the template
  isAvailableForNewContent: true
  screenshotPath: ../images/template-previews/screenshot-of-the-template.png
-->
{% extends "./layouts/base.hubl.html" %}

{% block header %}
  {% global_partial path="./partials/lp-header.hubl.html" type="HEADER" %}
{% endblock %}

{% block body %}
{% endblock body %}

{% block footer %}
  {% global_partial path="./partials/lp-footer.hubl.html" type="FOOTER %}
{% endblock %}
```

The reason for the difference is that we use a different header/footer partial on our landing pages. The landing page header/footer is simpler and doesn't include navigation. By default the base file includes our general website header/footer.

### DND Areas
A template is made up of [drag and drop HubL tags](mdc:https:/developers.hubspot.com/docs/reference/cms/hubl/tags/dnd-areas). For each template a DND area should be added within the `block_body`. This DND area defines where a user can drag and drop modules/sections around within the page area. This is what the template would look like with a DND area:
```
<!--
  templateType: page
  label: Name of the template
  isAvailableForNewContent: true
  screenshotPath: ../images/template-previews/screenshot-of-the-template.png
-->
{% extends "./layouts/base.hubl.html" %}

{% block body %}
  {% dnd_area "dnd_area"
    label="Main section"
  %}
  {% end_dnd_area %}
{% endblock body %}
```

Within the DND area tag, a template is made up of sections. Sections are full width pieces of a page that define smaller chunks of a page. These are typically broken up visually with a mix of spacing and/or colors. There are two ways that we can include sections in our template which include:

#### Theme Sections (Preferred)
Themes have section files which are meant to be reusable within both the page editor by users but also on templates by developers. The reusable sections act somewhat similarly to React components in that on the section file you can define context, which would be similar to props in React, that would then allow you to use that section on the template and edit the section's contents using context. You can learn more about section context [here](mdc:https:/developers.hubspot.com/docs/guides/cms/content/templates/drag-and-drop/sections). To include a pre-built theme section onto a template you would use the `include_dnd_partial` which you can learn more about [here](mdc:https:/developers.hubspot.com/docs/guides/cms/content/templates/drag-and-drop/sections#add-a-section-partial-to-a-template).

Theme sections are located inside the /sections/ directory and should be referenced with the path `../sections/section-name.hubl.html",`. These sections:
- Are specific to the website theme
- Should be used before considering net new sections on the template
- May be new and not used in other templates yet
- Are referenced with `../` in the path

Here is an example of a template that references theme sections in the DND area:
```
{% dnd_area "dnd_area"
  label="Main section"
%}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HubSpot/cms-elevate-theme-public](https://github.com/HubSpot/cms-elevate-theme-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
