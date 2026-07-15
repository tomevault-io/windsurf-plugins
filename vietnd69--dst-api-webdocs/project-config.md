---
trigger: always_on
description: This rule provides specific templates for documenting different types of DST script modules.
---

# DST API Documentation Templates

This rule provides specific templates for documenting different types of DST script modules.

## Component Template

Use this template for documenting components:

```markdown
---
id: [Id]
title: [Component Name]
description: Component responsible for [primary functionality]
sidebar_position: [Number]

last_updated: YYYY-MM-DD
build_version: 676042
change_status: stable
---

# [Component Name]

## Version History
| Build Version | Change Date | Change Type | Description |
|---------------|-------------|-------------|-------------|
| 675312 | 2023-11-15 | stable | Current version |

## Overview

The `[Component Name]` component provides [primary functionality]. It is typically used on [entity types] to enable [behavior/feature].

## Component Properties

### inst.[property_name]

**Type:** `[type]`

**Status:** `stable` | `added in build 675312` | `modified in build 675312` | `deprecated in build 675312`

**Description:** [Property description]

### inst.[property_name2]

**Type:** `[type]`

**Status:** `stable` | `added in build 675312` | `modified in build 675312` | `deprecated in build 675312`

**Description:** [Property description]

## Component Methods

### inst.components.[component_name]:[methodName](mdc:dst-api-webdocs/param1, param2)

**Status:** `stable` | `added in build 675312` | `modified in build 675312` | `deprecated in build 675312`

**Source:** [dst-scripts/components/[component_name].lua](mdc:dst-scripts/components/[component_name].lua)

**Description:**
[Method description verified against source code]

**Parameters:**
- `param1` ([type]): [Parameter description]
- `param2` ([type]): [Parameter description]

**Returns:**
- ([return_type]): [Return value description]

**Example:**
```lua
local entity = TheWorld.entity
entity.components.[component_name]:[methodName](mdc:dst-api-webdocs/"example", 5)
```

## Events

### "[event_name]"

**Status:** `stable` | `added in build 675312` | `modified in build 675312` | `deprecated in build 675312`

**Description:**
[When this event is triggered and what it signifies]

**Parameters:**
- `data.param1` ([type]): [Description]
- `data.param2` ([type]): [Description]

**Example:**
```lua
inst:ListenForEvent("[event_name]", function(inst, data)
    -- Handle event
end)
```

## Common Uses

[Explain common scenarios where this component is used]

## Related Components

- [[Related Component]](./[related-component].md): [Relationship description]
```

## Stategraph Template

Use this template for documenting stategraphs:

```markdown
---
id: [Id]
title: [Stategraph Name]
description: Stategraph for [entity/behavior description]
sidebar_position: [Number]

last_updated: YYYY-MM-DD
build_version: 676042
change_status: stable
---

# [Stategraph Name]

## Version History
| Build Version | Change Date | Change Type | Description |
|---------------|-------------|-------------|-------------|
| 675312 | 2023-11-15 | stable | Current version |

## Overview

The `[Stategraph Name]` defines the behavior and animations for [entity type]. It controls [primary behaviors].

## States

### "[state_name]"

**Status:** `stable` | `added in build 675312` | `modified in build 675312` | `deprecated in build 675312`

**Description:**
[State description and purpose]

**Entry Function:**
```lua
[Entry function code example]
```

**Timeline:**
[Timeline events, if any]

**Events:**
[Events that can occur in this state]

**Tags:**
- `[tag1]`: [Tag purpose]
- `[tag2]`: [Tag purpose]

**Transitions:**
- To `[next_state1]`: [Condition]
- To `[next_state2]`: [Condition]

### "[state_name2]"

[Similar structure for other states]

## Events

### "[event_name]"

**Status:** `stable` | `added in build 675312` | `modified in build 675312` | `deprecated in build 675312`

**Description:**
[Event purpose and when it's triggered]

**Handler:**
```lua
[Event handler code example]
```

## Common State Sequences

[Describe common sequences of state transitions]

## Related Stategraphs

- [[Related Stategraph]](./[related-stategraph].md): [Relationship description]
```

## Brain Template

Use this template for documenting brain behaviors:

```markdown
---
id: [Id]
title: [Brain Name]
description: AI brain for [entity type]
sidebar_position: [Number]

last_updated: YYYY-MM-DD
build_version: 676042
change_status: stable
---

# [Brain Name]

## Version History
| Build Version | Change Date | Change Type | Description |
|---------------|-------------|-------------|-------------|
| 675312 | 2023-11-15 | stable | Current version |

## Overview

The `[Brain Name]` defines the AI behavior for [entity type]. It prioritizes [primary behaviors/goals].

## Brain Structure

```lua
-- Brain structure example
BrainCommon.AddNode(
    WhileNode(function() 
        return [condition] 
    end, "Priority Node",
    PriorityNode({
        [Node examples...]
    }))
)
```

## Behaviors

### [Behavior Name]

**Status:** `stable` | `added in build 675312` | `modified in build 675312` | `deprecated in build 675312`

**Description:**
[Behavior description and purpose]

**Conditions:**
[When this behavior activates]

**Actions:**
[What actions the entity takes]

**Example:**
```lua
[Code example of this behavior node]
```

## Memory Variables


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vietnd69/dst-api-webdocs](https://github.com/vietnd69/dst-api-webdocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
