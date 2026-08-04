---
trigger: always_on
description: This document provides comprehensive information for AI agents and developers working with the `@gravity-ui/navigation` library. It serves as a reference for understanding the library's architecture, components, patterns, and best practices.
---

# @gravity-ui/navigation - Agent Guide

This document provides comprehensive information for AI agents and developers working with the `@gravity-ui/navigation` library. It serves as a reference for understanding the library's architecture, components, patterns, and best practices.

## Table of Contents

1. [Overview](#overview)
2. [Installation & Setup](#installation--setup)
3. [Core Concepts](#core-concepts)
4. [Main Components](#main-components)
5. [Component Patterns](#component-patterns)
6. [Styling & Theming](#styling--theming)
7. [TypeScript Types](#typescript-types)
8. [Common Usage Patterns](#common-usage-patterns)
9. [Best Practices](#best-practices)
10. [Testing](#testing)
11. [Project Structure](#project-structure)

## Overview

`@gravity-ui/navigation` is a React component library for building navigation interfaces, specifically designed for creating sidebar navigation systems (AsideHeader) and related navigation components. The library is part of the Gravity UI design system ecosystem.

### Key Features

- **AsideHeader**: Main sidebar navigation component with collapsible/expandable states
- **ActionBar**: Horizontal bar for organizing navigation elements, actions, and breadcrumbs
- **Drawer**: Sliding panel component for additional content
- **Footer/MobileFooter**: Page footer components for desktop and mobile
- **Settings**: Settings panel with grouping and filtering capabilities
- **MobileHeader**: Mobile-specific header component
- **HotkeysPanel**: Keyboard shortcuts display component

### Library Purpose

The library provides a complete navigation solution for web applications, with special focus on:

- Responsive design (desktop and mobile variants)
- Customizable theming via CSS variables
- Accessibility support
- TypeScript-first development
- Integration with Gravity UI UIKit components

## Installation & Setup

### Installation

```bash
npm install @gravity-ui/navigation
```

### Peer Dependencies

The library requires these peer dependencies to be installed:

```bash
npm install --save-dev \
  @gravity-ui/uikit@^7.2.0 \
  @gravity-ui/icons@^2.2.0 \
  @bem-react/classname@^1.6.0 \
  react@^19.0.0 \
  react-dom@^19.0.0
```

**Note**: For production, use `@gravity-ui/uikit@^7.2.0` or higher.

### Import Patterns

```typescript
// Main components
import {AsideHeader, ActionBar, Drawer, Footer} from '@gravity-ui/navigation';

// Advanced layout components
import {PageLayout, PageLayoutAside, AsideFallback} from '@gravity-ui/navigation';

// Sub-components
import {ActionBar} from '@gravity-ui/navigation';
// ActionBar.Section, ActionBar.Group, ActionBar.Item, ActionBar.Separator

import {Drawer, DrawerItem} from '@gravity-ui/navigation';
```

## Core Concepts

### Navigation States

The `AsideHeader` component has two main visual states:

- **Expanded**: Full navigation with icons and text labels
- **Collapsed/Compact**: Only icons visible, text hidden

State is managed via the `compact` prop and `onChangeCompact` callback.

### Component Structure

The library follows a hierarchical component structure:

```
AsideHeader
├── Logo
├── SubheaderItems (top section)
├── MenuItems (middle section - main navigation)
└── Footer (bottom section)
    └── FooterItems
```

### Content Rendering

There are two approaches to rendering content alongside `AsideHeader`:

1. **Simple**: Pass content via `renderContent` prop
2. **Advanced**: Use `PageLayout` and `PageLayoutAside` for better performance (supports code splitting)

## Main Components

### AsideHeader

The primary navigation component. Provides a collapsible sidebar with three main sections.

#### Basic Usage

```typescript
import {AsideHeader} from '@gravity-ui/navigation';
import {Icon} from '@gravity-ui/uikit';
import {Home} from '@gravity-ui/icons';

<AsideHeader
  logo={{
    icon: logoIcon,
    text: 'My App',
    href: '/',
  }}
  menuItems={[
    {
      id: 'home',
      title: 'Home',
      icon: Home,
      current: true,
      onItemClick: () => navigate('/'),
    },
  ]}
  compact={isCompact}
  onChangeCompact={setIsCompact}
  renderContent={({size}) => <MainContent />}
/>
```

#### Key Props

- `compact`: Boolean - Controls collapsed/expanded state
- `onChangeCompact`: Callback when state changes
- `menuItems`: Array of `MenuItem` objects
- `subheaderItems`: Items displayed under logo
- `renderFooter`: Function to render bottom section
- `renderContent`: Function to render main page content
- `headerDecoration`: Boolean - Highlights top section with background color
- `customBackground`: ReactNode - Custom background element
- `panelItems`: Array of `DrawerItem` props for drawer panels
- `onMenuItemsChanged`: Required for AllPages functionality
- `topAlert`: TopAlertProps - Alert banner above navigation

#### MenuItem Structure

```typescript
interface MenuItem {
  id: string;
  title: React.ReactNode;
  icon?: IconProps['data'];
  iconSize?: number | string;
  current?: boolean; // Selected/active state
  pinned?: boolean; // Cannot be hidden in AllPages
  hidden?: boolean; // Visibility in AllPages
  link?: string; // HTML href

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gravity-ui/navigation](https://github.com/gravity-ui/navigation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
