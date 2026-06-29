---
trigger: always_on
description: **egui-rad-builder** is a Rapid Application Development (RAD) GUI builder tool for the egui immediate-mode GUI framework. It allows developers to visually design user interfaces through drag-and-drop, then generates production-ready Rust code for egui-based applications.
---

# Claude.md - Project Analysis & Improvement Roadmap

## Project Overview

**egui-rad-builder** is a Rapid Application Development (RAD) GUI builder tool for the egui immediate-mode GUI framework. It allows developers to visually design user interfaces through drag-and-drop, then generates production-ready Rust code for egui-based applications.

**Current Version:** 0.1.10
**License:** MIT
**Status:** Active early development

---

## Design Inspiration & Reference Architecture

### Mobius-ECS Analysis ([saturn77/mobius-ecs](https://github.com/saturn77/mobius-ecs))

Mobius-ECS is a visual UI designer for Rust/egui built on Entity Component System principles. Key design elements to consider:

#### Architecture Patterns Worth Adopting

| Pattern | Mobius Approach | Opportunity for egui-rad-builder |
|---------|-----------------|----------------------------------|
| **ECS Foundation** | Uses `bevy_ecs` for modularity | Consider lightweight ECS for widget management, enabling dynamic component composition |
| **Signals/Slots** | `egui_mobius` for thread-safe communication | Add event-driven widget communication for preview interactivity |
| **Two-Tier Structure** | Designer app + Framework library | Separate core widget library from builder app for reusability |
| **Docking Integration** | `egui_dock` (0.17.0) as template layer | Already using panels; could adopt egui_dock for modular window architecture |

#### Notable Features to Consider

1. **Visual Alignment Tools** - Horizontal/vertical alignment with distribution controls and grid snapping (priority from Issue #15)
2. **Hot-Reload Support** - Instant UI updates during development
3. **Template System** - Declarative UI structure definitions
4. **Project Export** - Generates complete Cargo projects with dependencies (already implemented here)

#### Code Generation Approach (Mobius)
- Uses `syntect` for syntax highlighting in generated code preview
- Produces production-ready egui code from visual designs
- Integrates signals/slots patterns into generated code

### egui Demo Widget Gallery Analysis ([egui.rs](https://www.egui.rs))

The official egui demo showcases best practices for widget organization and UX patterns:

#### Widget Organization Patterns

**Consistent Configuration**:
- Builder pattern with chainable methods (`.with_date_button()`, `.on_hover_text()`)
- State fields for: enabled, visible, opacity, interactivity
- Hover text documentation on all interactive elements

**Grid-Based Layout**:
- Two-column grid: labels/docs on left, widgets on right
- Consistent spacing and striping for visual hierarchy
- `ui.scope_builder()` for grouped widget state management

**Progressive Disclosure**:
- CollapsingHeader for expandable sections
- Feature-gated optional components
- Conditional widget inclusion without layout disruption

#### Widget Categories (Official egui)

| Category | Widgets |
|----------|---------|
| **Text & Display** | Label, Hyperlink, Separator |
| **Input** | TextEdit (with hint text), Button, Link |
| **Selection** | Checkbox, RadioButton, SelectableLabel, ComboBox |
| **Numeric** | Slider (with suffix), DragValue (with speed config) |
| **Feedback** | ProgressBar (animated), Spinner |
| **Visual** | Color picker, Image, Image+Button combo |
| **Structure** | CollapsingHeader, Grid (configurable striping/spacing) |

#### UX Best Practices from egui Demo

1. **Scope Isolation** - Wrap widget groups with shared opacity/interactivity states
2. **Conditional Features** - Feature-gated components (e.g., chrono-dependent DatePicker)
3. **Snapshot Testing** - Multiple pixel densities and theme combinations
4. **Hover Documentation** - Consistent `.on_hover_text()` across all interactive elements

---

## Recent Changes (2025-12-27)

### Phase 5: Panel Tabs and UX Improvements
Implemented tabbed panel interface for better workspace organization:

**Right Panel Tabs:**
- Added "Inspector" and "Code" tabs to the right panel
- Click tabs to switch between widget properties and generated code
- Cleaner UI - no more cramped Inspector + Code in one scroll

**egui_dock Research (Deferred):**
- Researched `egui_dock` v0.18 for full docking system
- Compatible with egui 0.33, but requires significant refactoring
- Current tabbed approach provides 80% of UX benefit with 20% of complexity
- Full docking integration reserved for future if user demand exists

### Phase 6 Feature: Live Preview Mode
- **Preview mode toggle** - F5 or View menu to switch between Edit and Preview modes
- **Toolbar indicator** - Color-coded Edit/Preview button shows current mode
- **Selection handles hidden** - In preview mode, no selection boxes or resize handles
- **Widget interaction** - Interact with widgets (click buttons, type in text fields) in preview mode
- **Shortcuts updated** - Added F5 to palette shortcuts list

### Phase 4 Implementation (Code Generation)
- **Syntax highlighting** - Added `syntect` crate for Rust code highlighting in code preview

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timschmidt/egui-rad-builder](https://github.com/timschmidt/egui-rad-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
