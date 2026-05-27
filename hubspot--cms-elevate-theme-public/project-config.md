---
trigger: always_on
description: Configure template AI scaffolding
---

# Configure Template AI Scaffolding

After scaffolding a new template and setting up translations for it, follow these guidelines to set up a scaffolding schema file and pass in values from that schema file into the template. This should be focused on setting up the a scaffolding schema file and swapping the schema field values into the template file.

## Core Concepts

Scaffolding schema files allow us to provide context about certain pieces of content within a template. This is then used by several different AI tools within HubSpot such as content remix and AI website page builder which takes this context and context from the user about their business, in order to dynamically generate content for a given section that is more relevant to the user.

### Creating a scaffolding schema file
```
src/unified-theme/templates/schemas/[template-name].scaffold_schema.json
```

### Filling out the scaffolding schema file with fields

A scaffolding schema file is a JSON file that is comprised of fields and groups. Below is a brief description of each of the supported fields:

#### Groups
A group makes it easier to organize schema files in logical groups. Given templates can have a lot of content, we try to group relevant pieces of content. E.g. for a template that has a section with a heading and three cards, we'd likely create a top level group for the section that contains a heading field and three different card groups that then have a set of fields related to the card's content.

```
{
  "name": "group_card_one",
  "type": "group",
  "children": [
  ]
}
```

#### Text
The text field allows us to pass relevant context about what a piece of text should be about within the context of the template.

```
{
  "name": "card_heading",
  "type": "text",
  "description": "A heading that would be at the top of the first card in this section within the template."
}
```

#### Images
The image field allows us to pass relevant context about what an image should be about within the context of the template. The width:height ratio must be 1:1, 1:1.75, or 1.75:1.

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

### Adding a reference to the schema file in the template

To reference values from the newly created scaffolding schema file we would include a reference to the scaffolding schema file path in the template's template annotations via `scaffoldingSchemaPath`.

```
<!--
  templateType: page
  label: Landing page - signup
  isAvailableForNewContent: true
  scaffoldSchemaPath: ./schemas/lp-signup.scaffold_schema.json
  screenshotPath: ../images/template-previews/lp-signup.png
-->
```

### Passing in the value from the scaffolding schema file into the template

After setting up a path to the `scaffoldingSchemaPath`, we can now reference values from our schema file in our template file. Please note that these schema values will not always be populated so we'd want to set a fallback. You may notice that a template has a template_translations value passed in. The schema value should be passed first and then it shoudl fall back to content which we pull from template_translations. Below is an example:

```
{% dnd_row %}
  {% dnd_module
    path="../components/modules/Heading",
    headingAndTextHeadingLevel="h1",
    headingAndTextHeading={{ scaffold_content.heading or template_translations.lp_signup_hero_heading.message }},
    groupStyle={
      "alignment": {
        "text_align": "LEFT"
      },
      "headingStyleVariant": "display_2"
    }
  %}
  {% end_dnd_module %}
{% end_dnd_row %}
{% dnd_row %}
  {% dnd_module
    path="@hubspot/linked_image",
    img={
      "alt": scaffold_content.image.alt or "",
      "loading": "eager",
      "max_height": scaffold_content.image.height or 600,
      "max_width": scaffold_content.image.width or 1000,
      "size_type": "auto_custom_max",
      "src": scaffold_content.image.src or get_asset_url("../images/hero-banner.png")
    }
  %}
  {% end_dnd_module %}
{% end_dnd_row %}
```

## Reference Documentation
This is the [main Hubspot developer documentation](mdc:https:/developers.hubspot.com/docs/cms/building-blocks/content-scaffolding?hs_preview=XjcizpKB-164164808242#configure-scaffolding-schema)for setting up a scaffolding schema file.

## Implementation Steps

1. **Analyze Template Content to See Where Scaffolding Schema Should Be Added**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HubSpot/cms-elevate-theme-public](https://github.com/HubSpot/cms-elevate-theme-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
