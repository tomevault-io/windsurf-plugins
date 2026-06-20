---
trigger: always_on
description: Provides complete Discord Message Components V2 reference. Auto-invoke when
---

---
name: discord-components-v2
description: >
  Provides complete Discord Message Components V2 reference. Auto-invoke when
  the user is working with Discord bot components, message components, buttons,
  selects, modals, action rows, sections, containers, or Components V2 payloads.
---

# Discord: Message Components V2 (for LLMs)

Since 2025, message components got a lot more powerful, allowing for more interactive and dynamic user experiences.

## IS_COMPONENTS_V2 Flag

To use the new layout and content components (Section, Text Display, Thumbnail, Media Gallery, File, Separator, Container), send the message flag `1 << 15` (`32768`) as `flags` in your message payload. Once set on a message, this flag cannot be removed.

When `IS_COMPONENTS_V2` is active:

- `content` and `embeds` fields are disabled — use Text Display and Container instead
- Attachments must be exposed through components
- `poll` and `stickers` are disabled
- Messages allow up to **40 total components**

## Overview

- **Layout Components** — For organizing and structuring content (Action Row, Section, Container, Label)
- **Content Components** — For displaying static text, images, and files (Text Display, Thumbnail, Media Gallery, File)
- **Interactive Components** — For user interactions (Button, Select Menus, Text Input, File Upload, Radio Group, Checkbox Group, Checkbox)

### Component Types

| Type | Name                                      | Description                                                    | Style       | Usage          |
| ---- | ----------------------------------------- | -------------------------------------------------------------- | ----------- | -------------- |
| 1    | [Action Row](#action-row)                 | Container to display a row of interactive components           | Layout      | Message        |
| 2    | [Button](#button)                         | Button object                                                  | Interactive | Message        |
| 3    | [String Select](#string-select)           | Select menu for picking from defined text options              | Interactive | Message, Modal |
| 4    | [Text Input](#text-input)                 | Text input object                                              | Interactive | Modal          |
| 5    | [User Select](#user-select)               | Select menu for users                                          | Interactive | Message, Modal |
| 6    | [Role Select](#role-select)               | Select menu for roles                                          | Interactive | Message, Modal |
| 7    | [Mentionable Select](#mentionable-select) | Select menu for mentionables (users _and_ roles)               | Interactive | Message, Modal |
| 8    | [Channel Select](#channel-select)         | Select menu for channels                                       | Interactive | Message, Modal |
| 9    | [Section](#section)                       | Container to display text alongside an accessory component     | Layout      | Message        |
| 10   | [Text Display](#text-display)             | Markdown text                                                  | Content     | Message, Modal |
| 11   | [Thumbnail](#thumbnail)                   | Small image that can be used as an accessory                   | Content     | Message        |
| 12   | [Media Gallery](#media-gallery)           | Display images and other media                                 | Content     | Message        |
| 13   | [File](#file)                             | Displays an attached file                                      | Content     | Message        |
| 14   | [Separator](#separator)                   | Component to add vertical padding between other components     | Layout      | Message        |
| 17   | [Container](#container)                   | Container that visually groups a set of components             | Layout      | Message        |
| 18   | [Label](#label)                           | Container associating a label and description with a component | Layout      | Modal          |
| 19   | [File Upload](#file-upload)               | Component for uploading files                                  | Interactive | Modal          |
| 21   | [Radio Group](#radio-group)               | Single-choice set of options                                   | Interactive | Modal          |
| 22   | [Checkbox Group](#checkbox-group)         | Multi-selectable group of checkboxes                           | Interactive | Modal          |
| 23   | [Checkbox](#checkbox)                     | Single checkbox for yes/no choice                              | Interactive | Modal          |

## Anatomy of a Component

All components share these base fields:

| Field | Type    | Description                                                                                                          |
| ----- | ------- | -------------------------------------------------------------------------------------------------------------------- |
| type  | integer | The type of the component                                                                                            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [The-LukeZ/ComponentsV2ForLLMs](https://github.com/The-LukeZ/ComponentsV2ForLLMs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
