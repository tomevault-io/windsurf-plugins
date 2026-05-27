---
trigger: always_on
description: Configure section AI scaffolding
---

# Configure Section AI Scaffolding

After scaffolding a new section and setting up translations for it, follow these guidelines to set up a scaffolding schema file and pass in values from that schema file into the section. This should be focused on setting up the a scaffolding schema file and swapping the schema field values into the section file. 

## Core Concepts

Scaffolding schema files allow us to provide context about certain pieces of content within a section. This is then used by several different AI tools within HubSpot such as content remix and AI website page builder which takes this context and context from the user about their business, in order to dynamically generate content for a given section that is more relevant to the user. 

### Creating a scaffolding schema file
```
src/unified-theme/sections/schemas/[section-name].scaffold_schema.json
```

### Filling out the scaffolding schema file with fields

A scaffolding schema file is a JSON file that is comprised of fields and groups. Below is a brief description of each of the supported fields:

#### Groups
A group makes it easier to organize schema files in logical groups. Given sections can have a lot of content, we try to group relevant pieces of content. E.g. for a section that has a heading and three cards, we'd likely create a top level heading field and three different card groups that then have a set of fields related to the card's content.

```
{
  "name": "group_card_one",
  "type": "group",
  "children": [
  ]
}
```

#### Text 
The text field allows us to pass relevant context about what a piece of text should be about within the context of the section.

```
{
  "name": "card_heading",
  "type": "text",
  "description": "A heading that would be at the top of the first card in this section."
}
```

#### Images
The image field allows us to pass relevant context about what an image should be about within the context of the section. The width:height ratio must be 1:1, 1:1.75, or 1.75:1.

```
{
  "name": "card_image",
  "type": "image",
  "description": "An image that visually captures an element or aspect of the quote.",
  "width": "300",
  "height": "300"
}
```

#### Forms
The form field does not allow for an AI generated form. Instead it serves as a placeholder for the team that owns the AI landing page flow to create a form using an API and pass in its values using this field. 

```
{
  "name": "form",
  "type": "form",
  "description": "A form that a user would fill out to get access to an e-Book"
}
```

#### Meetings
The meeting field does not allow for an AI generated meeting link. Instead it serves as a placeholder for the team that owns the AI landing page flow to pass in a meeting link.

```
{
  "name": "meeting",
  "type": "meeting",
  "description": "A meeting link that will go in the banner of the page where someone can schedule a meeting with someone from the company."
}
```

### Adding a reference to the schema file in the section

To reference values from the newly created scaffolding schema file we would include a reference to the scaffolding schema file path in the section's template annotations via `scaffoldingSchemaPath`. 

```
<!--
  templateType: section
  label: About us
  isAvailableForNewContent: true
  scaffoldSchemaPath: ./schemas/about-us.scaffold_schema.json
  screenshotPath: ../images/section-previews/about-us.png
  description: "Section with an image on the left and company description on the right."
-->
```

### Passing in the value from the scaffolding schema file into the section

After setting up a path to the `scaffoldingSchemaPath`, we can now reference values from our schema file in our section file. Please note that these schema values will not always be populated (e.g. if a user adds one of these sections manually a schema value would not be generated) so we'd want to set a series of fallbacks. You may notice that a section has either a context variable being passed in, a template_translations value passed in, or a mix of both. Context should be referenced first as that is what is used if this section was included in a template and it should take precedence, followed by the schema value, and then falling back to content which we pull from template_translations in most cases. Below is an example:

```
{% dnd_row %}
  {% dnd_module
    path="../components/modules/Heading",
    headingAndTextHeadingLevel={{ context.heading.heading_level or "h1" }},
    headingAndTextHeading={{ context.heading.text or scaffold_content.heading or template_translations.section_hero_heading.message }},
    groupStyle={
      "alignment": {
        "text_align": "LEFT"
      },
      "headingStyleVariant": "display_2",
      "sectionStyleVariant": section_variant
    }
  %}
  {% end_dnd_module %}
{% end_dnd_row %}
{% dnd_row %}
  {% dnd_module
    path="@hubspot/linked_image",
    img={
      "alt": context.image.alt or scaffold_content.image.alt or "",
      "loading": context.image.loading or "eager",
      "max_height": context.image.height or scaffold_content.image.height or 600,
      "max_width": context.image.width or scaffold_content.image.width or 1000,
      "size_type": "auto_custom_max",
      "src": context.image.src or scaffold_content.image.src or get_asset_url("../images/hero-banner.png")
    }
  %}
  {% end_dnd_module %}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HubSpot/cms-elevate-theme-public](https://github.com/HubSpot/cms-elevate-theme-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
