---
trigger: always_on
description: This guide provides comprehensive instructions for AI agents on how to use the Mortar library to create iOS UI elements declaratively in Swift.
---

# Guide for AI Agents: Using Mortar Library for Swift UI Development

This guide provides comprehensive instructions for AI agents on how to use the Mortar library to create iOS UI elements declaratively in Swift.

## Overview

Mortar is a Swift DSL (Domain Specific Language) that enables declarative, anonymous view hierarchy construction using UIKit. It bridges the gap between traditional UIKit development and SwiftUI-like syntax while maintaining full compatibility with existing UIKit infrastructure.

## Key Features

1. **Anonymous View Construction**: Create complete view hierarchies without naming views or defining them outside of their usage context
2. **Declarative Layout**: Provides a clean syntax for AutoLayout constraints that works with UIKit's native classes
3. **Reactive Integration**: Seamlessly integrates with CombineEx for reactive programming patterns
4. **Managed Views**: Specialized components for UITableView and UICollectionView that work with model-driven data

## Core Concepts

### Result Builder Pattern
Mortar uses `MortarAddSubviewsBuilder` to enable anonymous view creation within UIKit's initialization blocks. This allows views to be created and added inline without explicit naming.

### Layout Properties
Views have access to layout properties that provide constraint capabilities:
- `layout`: Access to the view's own layout anchors
- `parentLayout`: Access to parent layout anchors for constraints
- `referencedLayout(_:)`: Access to referenced layout anchors for cross-view constraints

### Reactive Programming
Mortar integrates with CombineEx framework for reactive programming patterns:
- Event handling with `handleEvents()`
- Property binding with `bind()`
- Publisher sinking with `sink()`

## Basic Usage Patterns

### Creating Views with Anonymous Hierarchy

```swift
import Mortar

class MyViewController: UIViewController {
    override func loadView() {
        view = UIContainer {
            $0.backgroundColor = .darkGray

            UIVStack {
                $0.alignment = .center
                $0.backgroundColor = .lightGray
                $0.layout.sides == $0.parentLayout.sideMargins
                $0.layout.centerY == $0.parentLayout.centerY

                UILabel {
                    $0.layout.height == 44
                    $0.text = "Hello, World!"
                    $0.textColor = .red
                    $0.textAlignment = .center
                }

                UIButton(type: .roundedRect) {
                    $0.setTitle("Button", for: .normal)
                    $0.handleEvents(.touchUpInside) { NSLog("touched \($0)") }
                }
            }
        }
    }
}
```

### Layout Constraints

```swift
// Basic constraint against parent layout
$0.layout.centerY == $0.parentLayout.centerY

// Multi-constraint guide in single expression  
$0.layout.sides == $0.parentLayout.sideMargins

// Constraint to constants
$0.layout.size == CGSize(width: 100, height: 100)

// Inequalities
$0.layout.trailing == $0.parentLayout.trailing

// Constraint modification after creation
let group = $0.layout.center == $0.parentLayout.center
group.layoutConstraints.first?.constant += 20
```

### Reactive Programming

```swift
// Handle UIControl events with CombineEx Actions
$0.handleEvents(.valueChanged, model.toggleStateAction) { $0.isOn }

// Bind publishers to view properties
$0.bind(\.text) <~ model.toggleState.map { "Toggle is \($0)" }

// Sink publishers for complex view updates
$0.sink(model.someVoidPublisher) { view in
  // Void publishers handling
}

$0.sink(model.someValuePublisher) { view, value in
  // Value publishers handling
}
```

## Working with Managed Views

### Managed Table Views

```swift
// Define model and cell classes
private struct SimpleTextRowModel: ManagedTableViewCellModel {
    typealias Cell = SimpleTextRowCell

    let text: String
}

private final class SimpleTextRowCell: UITableViewCell, ManagedTableViewCell {
    typealias Model = SimpleTextRowModel
}

// Use in view controller
class BasicManagedTableViewController: UIViewController {
    override func loadView() {
        view = UIContainer {
            $0.backgroundColor = .white

            ManagedTableView {
                $0.layout.edges == $0.parentLayout.edges
                $0.sections <~ Property(value: [self.makeSection()])
            }
        }
    }
    
    private func makeSection() -> ManagedTableViewSection {
        ManagedTableViewSection(
            rows: [
                SimpleTextRowModel(text: "Simple row 1"),
                SimpleTextRowModel(text: "Simple row 2"), 
                SimpleTextRowModel(text: "Simple row 3"),
            ]
        )
    }
}
```

## Layout Guide Attributes

Mortar provides virtual attributes that represent multiple sub-attributes:

- **Position attributes**: `left`, `right`, `top`, `bottom`, `leading`, `trailing`, `centerX`, `centerY`
- **Size attributes**: `width`, `height`
- **Multi-attribute guides**:
  - `sides`: combines leading and trailing
  - `caps`: combines top and bottom
  - `size`: combines width and height
  - `edges`: combines top, leading, bottom, trailing
  - `center`: combines centerX and centerY


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmfieldman/Mortar](https://github.com/jmfieldman/Mortar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
