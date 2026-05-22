---
trigger: always_on
description: HTML documentation styling guide — design system, code highlighting, command blocks, section containers, and accessibility for project HTML docs.
---


# HTML Documentation Styling Guide and Design System

## Overview
This rule defines the standard styling patterns, design system, and best practices for creating HTML documentation files in the project. It ensures consistency, accessibility, and user-friendly presentation across all documentation.

## Problem Solved
- **Consistent Documentation Styling**: Ensures all HTML documentation follows the same visual patterns
- **Code Highlighting**: Standardizes code syntax highlighting with light blue background and dark blue text
- **Copy Button Functionality**: Provides consistent copy-to-clipboard functionality for commands **and for copy-paste prompts** (see [Copyable prompt blocks](#copyable-prompt-blocks))
- **Windows Compatibility**: Ensures commands are provided in single-line format for Windows users
- **Visual Hierarchy**: Creates clear visual distinction between different types of content
- **Accessibility**: Ensures proper contrast and readable text

## Core Design Principles

### Color Scheme
- **Code/Command Blocks**: Light blue background (`#e8f4f8`) with dark blue text (`#0d3b66`) and border (`#b8d4e3`)
- **Info Boxes**: Light blue background (`#d1ecf1`) with dark blue border (`#0c5460`)
- **Warning Boxes**: Light yellow background (`#fff3cd`) with yellow border (`#ffc107`)
- **Success Boxes**: Light green background (`#d4edda`) with green border (`#28a745`)
- **Error Boxes**: Light red background (`#f8d7da`) with red border (`#dc3545`)
- **Command Blocks**: Color-coded gradients based on script type (setup=blue, expedite=purple, etc.)

### Code Highlighting in Boxes
- **All code references** within info/warning/success/error boxes must use light blue background (`#e8f4f8`) with dark blue text (`#0d3b66`) and border (`#b8d4e3`)
- Use `<code class="code-highlight">` for inline code that needs highlighting
- Standard `<code>` tags inherit box background (use `code-highlight` class for light blue/dark blue styling)

### Script Name Highlighting
- **All script file names** (e.g., `setup-test-clock.js`, `expedite-stripe-renewal-test-clock.js`) must use the `script-name` class
- Script names should be visually distinct with purple gradient background
- Use `<code class="script-name">script-name.js</code>` for all script file references

### Section Introduction Containers
- **Major section introductions** (like "Using the Setup Test Clock Script") should use `section-intro` class
- Provides light blue gradient background with blue left border
- Wraps introductory paragraphs that explain what a script or section does

### Parameters List Styling
- **Parameters sections** must use `parameters-list` class
- Orange gradient background with orange left border
- All parameter names in code tags should use light blue background (`#e8f4f8`) with dark blue text (`#0d3b66`)

## Command Block Styling

### Color-Coded Command Blocks
Each script type has a distinct color scheme:

```css
/* Setup Scripts - Blue Gradient */
.command-block.setup {
    background: linear-gradient(135deg, #e3f2fd 0%, #bbdefb 100%);
    border-color: #2196f3;
}

/* Expedite Scripts - Purple Gradient */
.command-block.expedite {
    background: linear-gradient(135deg, #f3e5f5 0%, #e1bee7 100%);
    border-color: #9c27b0;
}

/* Advance Scripts - Orange Gradient */
.command-block.advance {
    background: linear-gradient(135deg, #fff3e0 0%, #ffe0b2 100%);
    border-color: #ff9800;
}

/* Verify Scripts - Green Gradient */
.command-block.verify {
    background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
    border-color: #4caf50;
}
```

### Command Block Structure with Parameter Customization
```html
<div class="command-block setup"
     data-command-type="setup"
     data-command-base="node scripts/path/to/script.js"
     data-template-command="node scripts/path/to/script.js --param1=value1 --param2=&quot;value2&quot;">
    <div class="command-header">
        <span class="command-type">Setup Script</span>
        <div class="command-header-buttons">
            <button class="copy-button-template" onclick="copyTemplateCommand(this, 'command-id')" title="Copy template command">📄 Template</button>
            <button class="copy-button" onclick="copyCommand(this, 'command-id')" title="Copy customized command">📋 Copy</button>
        </div>
    </div>
    <div class="command-parameters">
        <h5>🔧 Customize Parameters:</h5>
        <div class="parameter-group">
            <div class="parameter-input">
                <label for="command-id-param1">Parameter 1:</label>
                <input type="text" id="command-id-param1" placeholder="value1" value="default1" oninput="updateCommand('command-id', 'param1', this.value)">
            </div>
            <div class="parameter-input">
                <label for="command-id-param2">Parameter 2:</label>
                <input type="text" id="command-id-param2" placeholder="value2" value="default2" oninput="updateCommand('command-id', 'param2', this.value)">
            </div>
        </div>
        <div class="parameter-group single">
            <div class="parameter-input">
                <label for="command-id-param3">Parameter 3:</label>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
