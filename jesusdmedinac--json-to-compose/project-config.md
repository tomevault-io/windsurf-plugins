---
trigger: always_on
description: This document provides essential context for AI agents to collaborate effectively on this project.
---

# AGENTS.md - Guide for AI Agents

This document provides essential context for AI agents to collaborate effectively on this project.

## Overview

**json-to-compose** is a Server-Driven UI (SDUI) framework for Kotlin Multiplatform that converts JSON structures into Jetpack Compose components at runtime.

**Main use case:** The backend controls the application's UI without requiring app updates.

## Project Structure

```
json-to-compose/
├── library/       → Core library (published on Maven Central v1.0.3)
├── composy/       → Visual editor web/desktop
├── composeApp/    → Multiplatform demo app
├── server/        → Ktor Backend
└── shared/        → Shared utilities
```

### Main Module: `/library`

Location of core code:

```
library/src/commonMain/kotlin/com/jesusdmedinac/jsontocompose/
├── JsonToCompose.kt              → Entry point, CompositionLocals, router
├── model/
│   ├── ComposeNode.kt            → Serializable UI tree node
│   ├── ComposeType.kt            → Component types enum
│   ├── ComposeModifier.kt        → Serializable modifiers
│   └── NodeProperties.kt         → Component-specific props (sealed interface)
├── renderer/
│   ├── ColumnRenderer.kt         → ComposeNode.ToColumn()
│   ├── RowRenderer.kt            → ComposeNode.ToRow()
│   ├── BoxRenderer.kt            → ComposeNode.ToBox()
│   ├── TextRenderer.kt           → ComposeNode.ToText()
│   ├── ButtonRenderer.kt         → ComposeNode.ToButton()
│   ├── ImageRenderer.kt          → ComposeNode.ToImage()
│   ├── TextFieldRenderer.kt      → ComposeNode.ToTextField()
│   ├── LazyColumnRenderer.kt     → ComposeNode.ToLazyColumn()
│   ├── LazyRowRenderer.kt        → ComposeNode.ToLazyRow()
│   ├── ScaffoldRenderer.kt       → ComposeNode.ToScaffold()
│   ├── CardRenderer.kt           → ComposeNode.ToCard()
│   ├── AlertDialogRenderer.kt    → ComposeNode.ToAlertDialog()
│   ├── TopAppBarRenderer.kt      → ComposeNode.ToTopAppBar()
│   ├── CustomRenderer.kt         → ComposeNode.ToCustom()
│   ├── Alignment.kt              → String → Compose Alignment mappers
│   └── Arrangment.kt             → String → Compose Arrangement mappers
├── modifier/
│   └── ModifierMapper.kt         → Applies modifier operations
├── behavior/
│   └── Behavior.kt               → Interface for click events
└── state/
    └── StateHost.kt              → Interface for state management
```

## Core Architecture

### Data Flow

```
JSON String → kotlinx.serialization → ComposeNode → Router → Specific Renderer → Compose UI
```

### Main Router (`JsonToCompose.kt:34-47`)

```kotlin
@Composable
fun ComposeNode.ToCompose() {
    when (type) {
        ComposeType.Column -> ToColumn()
        ComposeType.Row -> ToRow()
        ComposeType.Box -> ToBox()
        ComposeType.Text -> ToText()
        ComposeType.Button -> ToButton()
        ComposeType.Image -> ToImage()
        ComposeType.TextField -> ToTextField()
        ComposeType.LazyColumn -> ToLazyColumn()
        ComposeType.LazyRow -> ToLazyRow()
        ComposeType.Scaffold -> ToScaffold()
        ComposeType.Card -> ToCard()
        ComposeType.AlertDialog -> ToAlertDialog()
        ComposeType.TopAppBar -> ToTopAppBar()
        ComposeType.Custom -> ToCustom()
    }
}
```

### Supported Components

| Type        | Props Class      | Category         |
| ----------- | ---------------- | ---------------- |
| Text        | TextProps        | Leaf             |
| Image       | ImageProps       | Leaf             |
| TextField   | TextFieldProps   | Leaf             |
| Button      | ButtonProps      | Single Child     |
| Scaffold    | ScaffoldProps    | Single Child     |
| Card        | CardProps        | Single Child     |
| Column      | ColumnProps      | Container        |
| Row         | RowProps         | Container        |
| Box         | BoxProps         | Container        |
| LazyColumn  | ColumnProps      | Container (lazy) |
| LazyRow     | RowProps         | Container (lazy) |
| AlertDialog | AlertDialogProps | Dialog           |
| TopAppBar   | TopAppBarProps   | Navigation       |
| Custom      | CustomProps      | Extensible       |

## How to Add a New Component

### Step 1: Add to `ComposeType` enum

File: `model/ComposeType.kt`

```kotlin
enum class ComposeType {
    // ... existing
    NewComponent;

    fun isLayout(): Boolean = when (this) {
        Column, Row, Box, NewComponent -> true  // if it is a layout
        else -> false
    }

    fun hasChild(): Boolean = when (this) {
        Button, NewComponent -> true  // if it has a single child
        else -> false
    }
}
```

### Step 2: Create Props in `NodeProperties`

File: `model/NodeProperties.kt`

```kotlin
@Serializable
@SerialName("NewComponentProps")
data class NewComponentProps(
    val property1: String? = null,
    val property2: Int? = null,
    val children: List<ComposeNode>? = null,  // if it is a container
    val child: ComposeNode? = null,            // if it is single child
) : NodeProperties
```

**Important:** Use `@SerialName` for correct JSON serialization.

### Step 3: Create the Renderer

File: `renderer/{Name}Renderer.kt` (one file per renderer)

```kotlin
@Composable
fun ComposeNode.ToNewComponent() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jesusdmedinac/json-to-compose](https://github.com/jesusdmedinac/json-to-compose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
