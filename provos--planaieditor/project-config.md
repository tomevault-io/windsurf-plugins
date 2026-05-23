---
trigger: always_on
description: `svelte-splitpanes` is a powerful Svelte component for creating resizable pane layouts. Here's how to make the most of it:
---

## Svelte-Splitpanes: Best Practices Manual

`svelte-splitpanes` is a powerful Svelte component for creating resizable pane layouts. Here's how to make the most of it:

### 1. Basic Setup

The core of the library consists of two main components: `<Splitpanes>` (the container) and `<Pane>` (the individual resizable sections).

A minimal setup looks like this:

```svelte
<script>
  import { Pane, Splitpanes } from 'svelte-splitpanes';
</script>

<Splitpanes style="height: 400px">
  <Pane>
    <div>Pane 1 Content</div>
  </Pane>
  <Pane>
    <div>Pane 2 Content</div>
  </Pane>
</Splitpanes>
```
This will create a vertical split by default. Always ensure your `<Splitpanes>` container has a defined height (or width, for horizontal layouts) for proper rendering.

---
### 2. Configuring `<Splitpanes>`

The `<Splitpanes>` component offers several props to customize its behavior:

* **`horizontal`**: (Boolean, default: `false`)
    * Set to `true` for a horizontal layout (panes stack vertically, splitters are horizontal).
    * Example: `<Splitpanes horizontal style="height: 300px">` 
* **`pushOtherPanes`**: (Boolean, default: `true`)
    * When `true`, dragging a splitter can push other splitters and resize adjacent panes beyond the immediate ones.
    * Set to `false` to lock the layout, meaning a splitter will only resize its direct neighbors and stop at their boundaries.
* **`dblClickSplitter`**: (Boolean, default: `true`)
    * Allows users to double-click a splitter to maximize the next pane.
    * Set to `false` to disable this feature.
* **`rtl`**: (Boolean | "auto", default: `"auto"`)
    * Enables Right-To-Left layout. `"auto"` will attempt to detect the direction from the computed style of the container.
* **`firstSplitter`**: (Boolean, default: `false`)
    * Set to `true` to display a splitter before the first pane. This allows maximizing the first pane via double-click but doesn't allow resizing it.
    * Example: `<Splitpanes {firstSplitter} {horizontal}>` 
* **`theme`**: (String, default: `'default-theme'`)
    * Apply a CSS class for styling the splitters. You can create custom themes.
    * Examples: `theme="no-splitter"`, `theme="modern-theme"`, `theme="my-theme"` 
* **`class`**: (String)
    * Add any additional CSS classes to the `<Splitpanes>` component.
* **`style`**: (String)
    * Apply inline styles, commonly used to set the height or width of the container.

---
### 3. Configuring `<Pane>`

Each `<Pane>` can be configured with the following props:

* **`size`**: (Number | null, default: `null`)
    * Sets the initial size of the pane in percentage.
    * If all panes have a `size` defined, their sum should ideally be 100.
    * If some panes have `size` and others don't, the remaining space is distributed among those without a defined size.
    * If no panes have a `size`, they will be distributed equally.
    * Can be two-way bound: `bind:size={paneSize}` for programmatic resizing.
* **`minSize`**: (Number, default: `0`)
    * Sets the minimum size of the pane in percentage.
    * Example: `<Pane minSize={20}>` 
* **`maxSize`**: (Number, default: `100`)
    * Sets the maximum size of the pane in percentage.
    * Example: `<Pane maxSize={70}>` 
* **`snapSize`**: (Number, default: `0`)
    * Defines a snap value in percentage. When resizing, if the pane's size gets close to its `minSize` plus `snapSize` (or `maxSize` minus `snapSize`), it can snap to that boundary.
    * Example: `<Pane snapSize={10}>` 
* **`class`**: (String)
    * Add any additional CSS classes to the `<Pane>` component.

---
### 4. Layout Management
* **Initial Sizes**: For a predictable initial layout, provide `size` props for your panes. Ensure the total percentage adds up to 100 for best results. If you omit `size` for one or more panes, they will share the remaining space. If all panes omit `size`, they will be divided equally.
    ```svelte
    <Splitpanes horizontal style="height: 400px">
      <Pane size={65}>65%</Pane>
      <Pane size={10}>10%</Pane>
      <Pane size={25}>25%</Pane>
    </Splitpanes>
    ```
* **Programmatic Resizing**: Use two-way binding with the `size` prop on a `<Pane>` to control its dimensions programmatically.
    ```svelte
    <script>
      let pane1Size = 30;
    </script>
    <Splitpanes>
      <Pane bind:size={pane1Size}>Dynamic Pane</Pane>
      <Pane>Second Pane</Pane>
    </Splitpanes>
    <input type="range" bind:value={pane1Size} min="10" max="90" />
    ```
    When one pane's size is changed programmatically, other panes without a specified size will adjust to accommodate the change.

---
### 5. Dynamic Panes (Adding, Removing, Reordering)

* **Adding/Removing Panes**: You can dynamically add or remove panes using Svelte's `#if` or `#each` blocks. `svelte-splitpanes` will automatically adjust the layout.
    ```svelte
    <script>
      let showSecondPane = true;
    </script>
    <Button on:click={() => showSecondPane = !showSecondPane}>Toggle Pane</Button>
    <Splitpanes>
      <Pane>Pane 1</Pane>
      {#if showSecondPane}
        <Pane>Pane 2 (Toggleable)</Pane>
      {/if}
      <Pane>Pane 3</Pane>
    </Splitpanes>
    ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [provos/planaieditor](https://github.com/provos/planaieditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
