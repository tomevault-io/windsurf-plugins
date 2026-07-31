---
trigger: always_on
description: > **Important**: The markdown extension is a early release and can be subject to change or may have edge cases that may not be supported yet. If you are encountering a bug or have a feature request, please open an issue on GitHub.
---





# Introduction into Markdown with Tiptap

> **Important**: The markdown extension is a early release and can be subject to change or may have edge cases that may not be supported yet. If you are encountering a bug or have a feature request, please open an issue on GitHub.

The Markdown extension provides bidirectional Markdown support for your Tiptap editor—parse Markdown strings into Tiptap's JSON format and serialize editor content back to Markdown.

## [](#core-capabilities)Core Capabilities

-   **Markdown Parsing**: Convert Markdown strings to Tiptap JSON
-   **Markdown Serialization**: Export editor content as Markdown
-   **Custom Tokenizers**: Add support for custom Markdown syntax
-   **Extensible Architecture**: Each extension can define its own parsing and rendering logic
-   **Utilities to Simplify Custom Syntax Creation**: `createBlockMarkdownSpec`, `createInlineMarkdownSpec` and more
-   **HTML Support**: Parse HTML embedded in Markdown using Tiptap's existing HTML parsing

## [](#how-it-works)How It Works

The Markdown extension acts as a bridge between Markdown text and Tiptap's JSON document structure.

It extends the base editor functionality by overwriting existing methods & properties with markdown-ready implementations, allowing for seamless integration between Markdown and Tiptap's rich text editor.

```
// Set initial content
const editor = new Editor({
  extensions: [StarterKit, Markdown],
  content: '# Hello World\n\nThis is **Markdown**!',
  contentType: 'markdown',
})

// Insert content
editor.commands.insertContent('# Hello World\n\nThis is **Markdown**!')
```

### [](#architecture)Architecture

```
Markdown String
      ↓
   MarkedJS Lexer (Tokenization)
      ↓
   Markdown Tokens
      ↓
   Extension Parse Handlers
      ↓
   Tiptap JSON
```

And in reverse:

```
Tiptap JSON
      ↓
   Extension Render Handlers
      ↓
   Markdown String
```

## [](#limitations)Limitations

The current implementation of the Markdown extension has some limitations:

-   **Comments are not supported yet**: Some advanced features like comments are not supported in Markdown. Be **cautious** when parsing Markdown content into a document that contains comments as they may be lost if replaced by Markdown content.
-   **Multiple child nodes in Tables**: Markdown tables are supported, but only one child node per cell is allowed as the Markdown syntax can't represent multiple child nodes.

## [](#why-markedjs)Why MarkedJS?

This extension integrates [MarkedJS](https://marked.js.org) as its parser:

-   **Fast and Lightweight**: One of the fastest Markdown parsers available
-   **Extensible**: Custom tokenizers enable non-standard Markdown syntax
-   **CommonMark Compliant**: Follows the CommonMark specification
-   **Battle-tested**: Widely used in production with active development

The Lexer API breaks Markdown into tokens that map naturally to Tiptap's node structure, making the integration clean and maintainable. The extension works identically in browser and server environments.



# Install and Setup the Markdown Package

This guide will walk you through installing and setting up the Markdown extension in your Tiptap editor.

## [](#installation)Installation

Install the Markdown extension using your preferred package manager:

```
npm install @tiptap/markdown
```

## [](#basic-setup)Basic Setup

Add the Markdown extension to your editor:

```
import { Editor } from '@tiptap/core'
import StarterKit from '@tiptap/starter-kit'
import { Markdown } from '@tiptap/markdown'

const editor = new Editor({
  element: document.querySelector('#editor'),
  extensions: [StarterKit, Markdown],
  content: '<p>Hello World!</p>',
})
```

That's it! Your editor now supports Markdown parsing and serialization.

### [](#initial-content-as-markdown)Initial Content as Markdown

To load Markdown content when creating the editor:

```
const editor = new Editor({
  extensions: [StarterKit, Markdown],
  content: '# Hello World\n\nThis is **Markdown**!',
  contentType: 'markdown',
})
```

## [](#configuration-options)Configuration Options

The Markdown extension accepts several configuration options:

### [](#indentation-style)Indentation Style

Configure how nested structures (lists, code blocks) are indented in the serialized Markdown:

```
Markdown.configure({
  indentation: {
    style: 'space', // 'space' or 'tab'
    size: 2, // Number of spaces or tabs
  },
})
```

**Examples:**

```
// Use 4 spaces for indentation (default: 2 spaces)
Markdown.configure({
  indentation: { style: 'space', size: 4 },
})

// Use tabs for indentation
Markdown.configure({
  indentation: { style: 'tab', size: 1 },
})
```

### [](#custom-marked-instance)Custom Marked Instance

If you need to use a custom version of marked or pre-configure it:

```
import { marked } from 'marked'

// Configure marked
marked.setOptions({
  gfm: true,
  breaks: true,
})

// Use custom marked instance
Markdown.configure({
  marked: marked,
})
```

### [](#marked-options)Marked Options

You can also pass marked options directly to the extension:

```
Markdown.configure({

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frappe/wiki](https://github.com/frappe/wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
