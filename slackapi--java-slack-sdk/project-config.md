---
trigger: always_on
description: Instructions for AI coding agents working on this module.
---

# AGENTS.md — slack-api-model

Instructions for AI coding agents working on this module.

## Module Overview

The `slack-api-model` module contains all core Slack data types: Block Kit blocks, elements, composition objects, event types, views, and other shared model objects. All other modules in the SDK depend on this one.

## Architecture

### Source Layout

```txt
src/main/java/com/slack/api/model/
├── block/                          # Block Kit UI framework types
│   ├── element/                    # Interactive elements (buttons, menus, inputs, etc.)
│   ├── composition/                # Reusable objects (text, options, confirmations, etc.)
│   ├── LayoutBlock.java            # Interface for all layout blocks
│   ├── BlockElement.java           # Abstract base for interactive elements (in element/)
│   ├── ContextBlockElement.java    # Interface for elements in context blocks
│   ├── ContextActionsBlockElement.java # Interface for elements in context actions blocks
│   ├── Blocks.java                 # Static factory helpers for layout blocks
│   ├── *Block.java                 # Concrete layout block types
│   └── Unknown*.java               # Fallback types for unknown JSON type values
├── event/                          # Slack event type definitions
├── view/                           # Modal and Home tab view types
└── ...                             # Other model types (File, Message, User, etc.)
```

### Key Design Principles

- **Gson deserialization via type factories** — Block Kit uses a manual discriminator pattern. Each category has a Gson factory (`GsonLayoutBlockFactory`, `GsonBlockElementFactory`, etc.) that reads the JSON `"type"` field and maps it to the concrete Java class. Unknown types fall through to `Unknown*` classes instead of throwing.
- **Lombok annotations** — All model types use `@Data`, `@Builder`, `@NoArgsConstructor`, `@AllArgsConstructor`. Layout blocks don't need `@EqualsAndHashCode(callSuper = false)` (they implement an interface), but elements do (they extend an abstract class).
- **Static factory helpers** — `Blocks.java` and `BlockElements.java` provide convenience methods for constructing types via the `ModelConfigurator` lambda pattern.

## Adding a New Block Kit Type

Block Kit types span three categories. The steps differ slightly for each:

| Category | Location | Base Type | Gson Factory |
|----------|----------|-----------|--------------|
| Layout blocks | `block/*.java` | `LayoutBlock` (interface) | `GsonLayoutBlockFactory` |
| Block elements | `block/element/*.java` | `BlockElement` (abstract class) | `GsonBlockElementFactory` |
| Composition objects | `block/composition/*.java` | N/A (standalone `@Data` classes) | `GsonTextObjectFactory` (text objects only) |

### Steps

#### 1. Look up the Block Kit documentation

Check the Slack API reference:
- Layout blocks: `https://docs.slack.dev/reference/block-kit/blocks/{block-type}`
- Elements: `https://docs.slack.dev/reference/block-kit/block-elements/{element-type}`
- Composition objects: `https://docs.slack.dev/reference/block-kit/composition-objects/{object-type}`

#### 2. Create the Java class

##### For a layout block

File: `src/main/java/com/slack/api/model/block/{Name}Block.java`

```java
package com.slack.api.model.block;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * https://docs.slack.dev/reference/block-kit/blocks/{type}-block
 */
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class CardBlock implements LayoutBlock {
    public static final String TYPE = "card";
    private final String type = TYPE;

    private String blockId;

    // Add fields from the API docs (camelCase in Java, auto-mapped to snake_case JSON)
}
```

For container blocks (those whose primary purpose is holding child elements), initialize the list with `@Builder.Default`:

```java
@Builder.Default
private List<ContextActionsBlockElement> elements = new ArrayList<>();
```

##### For a block element

File: `src/main/java/com/slack/api/model/block/element/{Name}Element.java`

```java
package com.slack.api.model.block.element;

import com.slack.api.model.block.composition.PlainTextObject;
import com.slack.api.model.block.composition.ConfirmationDialogObject;
import lombok.*;

/**
 * https://docs.slack.dev/reference/block-kit/block-elements/{type}-element
 */
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode(callSuper = false)
public class DatePickerElement extends BlockElement {
    public static final String TYPE = "datepicker";
    private final String type = TYPE;

    private String actionId;
    private PlainTextObject placeholder;
    private String initialDate;
    private ConfirmationDialogObject confirm;
    private Boolean focusOnLoad;
}
```

##### Key conventions

- The `TYPE` constant must match exactly what the Slack API returns in the `"type"` JSON field.
- Java fields are camelCase. Gson's `LOWER_CASE_WITH_UNDERSCORES` policy handles snake_case mapping automatically.
- **List initialization depends on the field's role:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slackapi/java-slack-sdk](https://github.com/slackapi/java-slack-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
