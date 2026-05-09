---
trigger: always_on
description: DatastarUI is a Go/templ port of shadcn/ui components that maintains pixel-perfect visual and behavioral parity while eliminating JavaScript dependencies (except for the 15KB Datastar library for reactivity).
---

# DatastarUI Component Development Guide

## Project Overview

DatastarUI is a Go/templ port of shadcn/ui components that maintains pixel-perfect visual and behavioral parity while eliminating JavaScript dependencies (except for the 15KB Datastar library for reactivity).

### Key Features

- **Pixel-perfect shadcn/ui components** ported to Go/templ
- **Globally Scoped Datastar signals** for interactive components. Signals are global on the page & scoped using `props.ID`. Datastar evaluated signals in the order they appear on the page.
- **Simple Usage Examples** signal creation complexity is managed by the DatastarUI components. The usage examples do not use `data-signals` attributes.

### Project Structure

```
datastarui/
├── components/                    # Reusable UI components
│   ├── button/
│   │   ├── button.templ          # Component template
│   │   ├── props.go              # Props and types
│   │   └── variants.go           # CSS variants
│   ├── form/                     # Form components
│   ├── input/                    # Input component
│   └── ...                       # Other components
├── pages/
│   ├── components/               # Component demo pages
│   │   ├── buttonpage/
│   │   │   └── button_page.templ # Button demo page
│   │   └── ...                   # Other demo pages
│   ├── home_page.templ           # Home page
│   └── docs_page.templ           # Documentation page
├── layouts/                      # Page layouts and navigation
├── static/                       # Static assets (CSS, JS)
└── main.go                       # Server and routing
```

## IMPORTANT

The developer is running a live reload server wathcing for file changes. Do not try to run the compiled binary as it is already running. Templ files will be automatically generated, but feel free to run `templ generate` to check for errors.

Other than checking for Templ compilation errors, do not try to check the results yourself. The developer will check the results and give you screen shots if needed. Do not run `go run main.go` do not run `go build`.

### Tailwind CSS Watch Process

**IMPORTANT**: Tailwind CSS is running in watch mode during development. The developer has `tailwindcss --watch` running automatically, which means:

- CSS changes in `static/css/index.css` are automatically compiled to `static/css/build.css`
- The watch process monitors all `.templ` files for class changes
- Only run `tailwind` or similar commands if there are CSS compilation issues that need debugging

## The Datastar Way - Best Practices

### Stop Overcomplicating It

Most of the time, if you run into issues when using Datastar, you are probably **overcomplicating it™**.

Datastar is a **hypermedia framework**, not a JavaScript framework. If you approach it like a JavaScript framework, you are likely to run into complications.

### The Hypermedia Approach

Between attribute plugins and action plugins, Datastar provides everything you need to build hypermedia-driven applications. Using this approach:

- **Backend drives state** to the frontend and acts as the single source of truth
- **Server determines** what actions the user can take next
- **State flows down through props, events flow up** - always encapsulate state and send props down, events up


**Alternative: Using `data-bind` with web components:**

```go
// Binding directly to web component value
<input data-bind-foo />
<my-component
    data-attr-src="$foo"
    data-bind-result
></my-component>
<span data-text="$result"></span>
```

```javascript
class MyComponent extends HTMLElement {
  static get observedAttributes() {
    return ["src"];
  }

  attributeChangedCallback(name, oldValue, newValue) {
    this.value = `You entered ${newValue}`;
    this.dispatchEvent(new Event("change")); // Required for data-bind
  }
}

customElements.define("my-component", MyComponent);
```

### Key Principles

1. **Think hypermedia first** - let the server drive state and available actions
2. **Use data-\* attributes** for all reactive behavior when possible
3. **Extract complex logic** into external scripts or web components
4. **Follow props down, events up** - encapsulate functionality and communicate via well-defined interfaces
5. **Avoid JavaScript framework patterns** - resist the urge to manage state in the frontend
6. **Keep it simple** - if it feels complicated, you're probably overengineering it

### DatastarUI Component Guidelines

When building DatastarUI components:

- **Prefer server-driven state** over complex client-side logic
- **Use Datastar expressions** for simple reactive behavior
- **Extract complex interactions** into web components when needed
- **Follow the hypermedia mindset** - components should be declarative and server-controlled
- **Test with minimal JavaScript** - the goal is to eliminate JavaScript dependencies while maintaining full functionality

# Go/templ Patterns

- **Use templ.Attributes** for flexible HTML attribute passing

```go
templ RefExample(props RefExampleProps) {
    {{
        refName := props.ID + "_ref"
        refAttrName := "data-ref-" + refName

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CoreyCole/datastarui](https://github.com/CoreyCole/datastarui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
