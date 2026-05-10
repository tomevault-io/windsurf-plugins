---
trigger: always_on
description: This document provides a comprehensive guide to all input types available in Langflow. Each input type is designed for specific use cases and data formats.
---

# Langflow Input Types Documentation

This document provides a comprehensive guide to all input types available in Langflow. Each input type is designed for specific use cases and data formats.

## Table of Contents

1. [Text Inputs](#text-inputs)
2. [Numeric Inputs](#numeric-inputs)
3. [Boolean Inputs](#boolean-inputs)
4. [Selection Inputs](#selection-inputs)
5. [Data Structure Inputs](#data-structure-inputs)
6. [File and Code Inputs](#file-and-code-inputs)
7. [Specialized Inputs](#specialized-inputs)
8. [Handle/Connector Inputs](#handleconnector-inputs)

---

## Text Inputs

### StrInput

**Field Type:** `"str"`

A basic text input field for single-line text entry.

**Key Features:**
- Supports single string or list of strings
- Can load values from database (`load_from_db`)
- Validates string type
- Supports metadata tracing

**Common Fields:**
- `name` (required): Field name
- `value` (default: `""`): Text value
- `placeholder`: Placeholder text
- `required`: Whether field is required
- `load_from_db`: Enable database loading
- `is_list`: Support list of strings

**Example:**
```python
StrInput(
    name="username",
    display_name="Username",
    placeholder="Enter your username",
    required=True
)
```

---

### MessageInput

**Field Type:** `"str"`  
**Input Types:** `["Message"]`

A text input specifically designed for message objects. Accepts Message objects, strings, or async iterators.

**Key Features:**
- Converts strings/iterators to Message objects
- Handles Message objects from different modules
- Supports input tracing

**Common Fields:**
- `name` (required): Field name
- `value`: Message object, string, or iterator
- `input_types`: `["Message"]` (automatic)

**Example:**
```python
MessageInput(
    name="user_message",
    display_name="User Message",
    value="Hello, world!"
)
```

---

### MessageTextInput

**Field Type:** `"str"`  
**Input Types:** `["Message"]`

A text input for messages with enhanced tracking capabilities.

**Key Features:**
- Extracts text from Message objects
- Supports Data objects with text_key
- Handles async iterators
- Metadata and input tracing enabled

**Common Fields:**
- `name` (required): Field name
- `value`: Message, string, Data object, or iterator
- `input_types`: `["Message"]` (automatic)

---

### MultilineInput

**Field Type:** `"str"`

A multi-line text input field with support for AI assistance.

**Key Features:**
- Multi-line text editing
- AI-enabled assistance (optional)
- Copy field functionality
- Inherits from MessageTextInput

**Common Fields:**
- `name` (required): Field name
- `value`: Multi-line text string
- `multiline`: `True` (default)
- `ai_enabled`: Enable AI assistance
- `copy_field`: Enable copy functionality

**Example:**
```python
MultilineInput(
    name="description",
    display_name="Description",
    placeholder="Enter a detailed description...",
    multiline=True,
    ai_enabled=True
)
```

---

### MultilineSecretInput

**Field Type:** `"str"` (password)

A multi-line secret/password input field with hidden text.

**Key Features:**
- Multi-line password input
- Text is hidden/masked
- Never tracked in telemetry
- Input tracing enabled

**Common Fields:**
- `name` (required): Field name
- `value`: Secret text
- `multiline`: `True` (default)
- `password`: `True` (default)
- `track_in_telemetry`: `False` (automatic)

**Example:**
```python
MultilineSecretInput(
    name="api_key",
    display_name="API Key",
    placeholder="Enter your API key..."
)
```

---

### SecretStrInput

**Field Type:** `"str"` (password)

A single-line password/secret input field.

**Key Features:**
- Password field with hidden text
- Loads from database by default
- Never tracked in telemetry
- Validates string, Message, Data, or iterator types

**Common Fields:**
- `name` (required): Field name
- `value`: Secret string
- `password`: `True` (default)
- `load_from_db`: `True` (default)
- `track_in_telemetry`: `False` (automatic)

**Example:**
```python
SecretStrInput(
    name="password",
    display_name="Password",
    placeholder="Enter password"
)
```

---

### QueryInput

**Field Type:** `"query"`

A specialized text input for search queries with optional separator support.

**Key Features:**
- Designed for search/query operations
- Optional separator for query parsing
- Inherits from MessageTextInput
- Supports query-specific formatting

**Common Fields:**
- `name` (required): Field name
- `value`: Query string
- `separator`: Optional separator character (e.g., `","`, `" "`)

**Example:**
```python
QueryInput(
    name="search_query",
    display_name="Search Query",
    separator=" ",
    placeholder="Enter search terms..."
)
```

---

## Numeric Inputs

### IntInput

**Field Type:** `"int"`

An integer number input field with range validation support.

**Key Features:**
- Validates integer values
- Converts floats to integers
- Supports range specifications (min/max)
- Supports list of integers
- Tracked in telemetry (safe numeric parameter)

**Common Fields:**
- `name` (required): Field name
- `value`: Integer value
- `range_spec`: RangeSpec object for min/max validation
- `is_list`: Support list of integers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Empreiteiro/langflow-factory](https://github.com/Empreiteiro/langflow-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
