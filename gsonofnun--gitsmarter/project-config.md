---
trigger: always_on
description: |IMPORTANT: Prefer retrieval-led reasoning over pre-training-led reasoning
---

# Widget UI Framework | [widgets Index]|root: ./src/widgets
|IMPORTANT: Prefer retrieval-led reasoning over pre-training-led reasoning
|Core:{ui_widgets_core.cpp:Widget,EventDispatcher,WidgetArena,FlexContainer}
|Basic:{ui_widgets_basic.cpp:Label,Button,TextInput,Checkbox}
|List:{ui_widgets_list.cpp:FileList,BranchTree,virtualized}
|Dialog:{ui_widgets_dialog.cpp:Dialog,ScrollContainer,ProgressBar}
|Diff:{ui_widgets_diff.cpp:DiffLineWidget,DiffViewerWidget,virtualized}
|Commit:{ui_widgets_commit.cpp:CommitRowWidget,CommitHistoryPanelWidget,virtualized}
|Sidebar:{ui_widgets_sidebar.cpp:SidebarWidget,SplitHandle,DragHandle}
|Titlebar:{ui_widgets_titlebar.cpp:TitleBarWidget,TitleBarButtonWidget}
|ContextMenu:{ui_widgets_context_menu.cpp:ContextMenuWidget,MenuItem}
|Welcome:{ui_widgets_welcome.cpp:LogoWidget,OutlineButton}

Custom Direct2D widget system for GitSmarter UI.

## Quick Patterns (Retrieval-Led)

**Creating a widget:**
```cpp
MyWidget* w = g_widget_arena.create<MyWidget>();
widget_add_child(parent, w);
w->flags |= WidgetFlags::Focusable;  // For keyboard interaction
```

**Widget lifecycle:** `measure()` → `layout()` → `render()`

**Event handling:** Return `true` to consume, `false` to bubble

**Dirty/render:** Call `set_dirty()` after state changes

## File Responsibilities

| File | Purpose |
|------|---------|
| ui_widgets_core.cpp | Widget base class, WidgetArena, EventDispatcher, FlexContainer |
| ui_widgets_basic.cpp | Label, Button, TextInput, Checkbox, SectionHeader |
| ui_widgets_list.cpp | FileList, BranchTree, StashList (virtualized) |
| ui_widgets_dialog.cpp | Dialog, ScrollContainer, ProgressBar |
| ui_widgets_sidebar.cpp | SidebarWidget, SplitHandle, DragHandle |
| ui_widgets_diff.cpp | DiffLineWidget, DiffViewerWidget (virtualized) |
| ui_widgets_commit.cpp | CommitRowWidget, CommitHistoryPanelWidget (virtualized) |
| ui_widgets_welcome.cpp | LogoWidget, OutlineButton, RecentRepoItemWidget |
| ui_widgets_titlebar.cpp | TitleBarWidget, TitleBarButtonWidget |
| ui_widgets_context_menu.cpp | ContextMenuWidget, MenuItem (popup context menus) |

## Core Widget Structure

```cpp
struct Widget {
    WidgetId id;
    const char* debug_name = nullptr;
    Widget* parent = nullptr;
    Widget* first_child = nullptr;
    Widget* last_child = nullptr;
    Widget* next_sibling = nullptr;
    Widget* prev_sibling = nullptr;
    uint16_t flags = WidgetFlags::Visible | WidgetFlags::Enabled;
    LayoutRect rect = {};
    float preferred_width = 0.0f, preferred_height = 0.0f;
    float padding_left/right/top/bottom = 0.0f;
    SizeMode width_mode = SizeMode::Hug;
    SizeMode height_mode = SizeMode::Hug;
    float flex = 0.0f;

    virtual void measure(const LayoutConstraints& constraints);
    virtual void layout();
    virtual void render(RenderContext& ctx);
    virtual bool on_mouse_enter/leave/move/down/up/wheel(x, y, ...);
    virtual bool on_key_down/up(vk, ctrl, shift, alt);
    virtual bool on_char(wchar_t ch);
    virtual bool on_focus/blur();
};
```

## Widget Flags Reference

| Flag | Value | Purpose |
|------|-------|---------|
| Visible | 0x0001 | Widget is rendered |
| Enabled | 0x0002 | Widget responds to input |
| Focusable | 0x0004 | Can receive keyboard focus |
| Focused | 0x0008 | Currently has focus (set by EventDispatcher) |
| Hovered | 0x0010 | Mouse is over widget |
| Pressed | 0x0020 | Mouse button down |
| Dirty | 0x0040 | Needs re-render |
| LayoutDirty | 0x0080 | Needs re-layout |
| ClipChildren | 0x0100 | Apply clip rect to children |
| CapturesMouse | 0x0200 | Receives all mouse events during drag |
| RendersOwnChildren | 0x0400 | Widget renders children manually (virtualization) |
| SkipTabStop | 0x0800 | Excluded from Tab navigation |

## Key Patterns

### Arena Allocation
All widgets from `g_widget_arena`:
```cpp
Button* btn = g_widget_arena.create<Button>();
```

### Measure Implementation
```cpp
void measure(const LayoutConstraints& constraints) override {
    // Calculate preferred_width/preferred_height based on content
    preferred_width = ...;
    preferred_height = ...;
    Widget::measure(constraints);  // Apply constraints (REQUIRED)
}
```

### Render Implementation
```cpp
void render(RenderContext& ctx) override {
    // Background with state-based coloring
    uint32_t bg = Theme::BG_SECONDARY;
    if (flags & WidgetFlags::Pressed) bg = Theme::ACCENT_PRESSED;
    else if (flags & WidgetFlags::Hovered) bg = Theme::BG_HOVER;
    else if (flags & WidgetFlags::Focused) bg = Theme::ACCENT;

    D2D1_RECT_F bg_rect = D2D1::RectF(rect.x, rect.y,
        rect.x + rect.width, rect.y + rect.height);
    ctx.fill_rounded_rect(bg_rect, Theme::CORNER_RADIUS, bg);

    // Focus ring
    if (flags & WidgetFlags::Focused) {
        ctx.brush->SetColor(color_from_argb(Theme::ACCENT));
        D2D1_ROUNDED_RECT rr = { bg_rect, Theme::CORNER_RADIUS, Theme::CORNER_RADIUS };
        ctx.target->DrawRoundedRectangle(rr, ctx.brush, 1.5f);
    }
}
```

### Mouse Event Handling
```cpp
bool on_mouse_down(float x, float y, int button) override {
    if (button != 0) return false;  // Only left click
    return true;  // Consume event
}

bool on_mouse_up(float x, float y, int button) override {
    if ((flags & WidgetFlags::Hovered) && on_click) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GSonofNun/GitSmarter](https://github.com/GSonofNun/GitSmarter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
