---
trigger: always_on
description: This document provides detailed information about all agents (skills) available in the Element Plus Skills Library.
---

# Element Plus Skills - Agents Documentation

This document provides detailed information about all agents (skills) available in the Element Plus Skills Library.

## 📋 Agent Index

| Category | Skills Count | Description |
|----------|--------------|-------------|
| [Basic Components](#basic-components-agents) | 14 | UI elements for basic interactions |
| [Form Components](#form-components-agents) | 24 | Form inputs and data collection |
| [Data Display](#data-display-agents) | 17 | Data visualization and presentation |
| [Navigation](#navigation-agents) | 8 | Navigation and routing components |
| [Feedback](#feedback-agents) | 12 | User feedback and notifications |
| [Layout](#layout-agents) | 4 | Page layout and structure |
| [Utility](#utility-agents) | 5 | Utility and helper components |
| [Design Specs](#design-specifications-agents) | 5 | Design system specifications |
| [Foundation](#foundation-agents) | 6 | Core setup and configuration |

---

## Basic Components Agents

### el-affix

**Description**: Fixes elements to a specific visible area for sticky navigation.

**Use Cases**:
- Sticky navigation headers
- Fixed toolbars while scrolling
- Persistent action buttons
- Floating sidebars

**Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| offset | number | 0 | Offset distance from top/bottom |
| position | string | 'top' | Position: 'top' or 'bottom' |
| target | string | - | Target container CSS selector |
| z-index | number | 100 | z-index of affix element |

**Example**:
```vue
<el-affix :offset="120">
  <el-button type="primary">Fixed Button</el-button>
</el-affix>
```

---

### el-alert

**Description**: Displays important alert messages on the page.

**Use Cases**:
- System notifications
- Status messages
- Warning displays
- Important information

**Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| title | string | - | Alert title (required) |
| type | string | 'info' | Type: primary, success, warning, info, error |
| description | string | - | Descriptive text |
| closable | boolean | true | Whether alert can be dismissed |
| show-icon | boolean | false | Whether to display type icon |

**Example**:
```vue
<el-alert title="Success" type="success" description="Operation completed" show-icon />
```

---

### el-anchor

**Description**: Anchor navigation for quick page section navigation.

**Use Cases**:
- Table of contents
- Document navigation
- Section anchors
- Quick page navigation

**Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| container | string/HTMLElement | - | Scroll container |
| offset | number | 0 | Scroll offset |
| direction | string | 'vertical' | Direction: 'vertical' or 'horizontal' |

**Example**:
```vue
<el-anchor>
  <el-anchor-link href="#section1" title="Section 1" />
  <el-anchor-link href="#section2" title="Section 2" />
</el-anchor>
```

---

### el-avatar

**Description**: Displays user avatars with images, icons, or characters.

**Use Cases**:
- User profile pictures
- Team member displays
- Entity representations
- Avatar groups

**Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| src | string | - | Image source URL |
| icon | string/Component | - | Icon component |
| size | number/string | - | Size: number or 'large'/'default'/'small' |
| shape | string | - | Shape: 'circle' or 'square' |

**Example**:
```vue
<el-avatar :size="50" src="avatar.jpg" />
<el-avatar :icon="UserFilled" />
```

---

### el-backtop

**Description**: Back-to-top button for long page navigation.

**Use Cases**:
- Long page navigation
- Quick scroll to top
- Improved user experience

**Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| target | string | - | Target to trigger scroll |
| visibility-height | number | 200 | Min height to show button |
| right | number | 40 | Right distance |
| bottom | number | 40 | Bottom distance |

**Example**:
```vue
<el-backtop :bottom="100" />
```

---

### el-badge

**Description**: Displays numbers or status marks on buttons and icons.

**Use Cases**:
- Notification counts
- Status indicators
- Unread message badges
- Attention markers

**Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| value | string/number | - | Display value |
| max | number | 99 | Maximum value |
| is-dot | boolean | false | Display as a dot |
| type | string | 'danger' | Badge type |

**Example**:
```vue
<el-badge :value="12">
  <el-button>Comments</el-button>
</el-badge>
```

---

### el-breadcrumb

**Description**: Displays page location for easier navigation.

**Use Cases**:
- Page navigation paths
- Category hierarchies
- Document structures

**Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| separator | string | '/' | Separator character |
| separator-icon | string/Component | - | Icon for separator |

**Example**:
```vue
<el-breadcrumb separator="/">
  <el-breadcrumb-item :to="{ path: '/' }">Home</el-breadcrumb-item>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jiaiyan/element-plus-skills](https://github.com/jiaiyan/element-plus-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
