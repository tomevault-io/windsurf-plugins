---
trigger: always_on
description: LaunchpadPlus is a modern macOS application launcher built with SwiftUI that serves as a replacement for Apple's deprecated Launchpad. It provides a beautiful, customizable interface for organizing and launching applications with advanced features like folders, search, and drag-and-drop functionality.
---

# LaunchpadPlus - Copilot Context

## Project Overview

LaunchpadPlus is a modern macOS application launcher built with SwiftUI that serves as a replacement for Apple's deprecated Launchpad. It provides a beautiful, customizable interface for organizing and launching applications with advanced features like folders, search, and drag-and-drop functionality.

## Architecture & Key Features

### 🎨 **Design Philosophy**
- **Glass Morphism UI**: Translucent, blurred backgrounds using `NSVisualEffectView`
- **Smooth Animations**: Fluid transitions with spring effects throughout
- **Justified Grid**: Icons and folders evenly distributed, filling all available space
- **Responsive Layout**: Adapts to any screen size and aspect ratio
- **Dark/Light Mode**: Automatic theme adaptation using SwiftUI's `colorScheme`

### 🏗️ **Architecture Pattern**
- **MVVM**: Model-View-ViewModel architecture using SwiftUI's `@StateObject` and `@ObservableObject`
- **Singleton Managers**: Centralized state management with `SettingsManager` and `AppManager`
- **Reactive UI**: SwiftUI bindings for real-time updates across views
- **Persistent Storage**: UserDefaults for settings and layout persistence

## Core Components

### 📱 **Main Application Structure**
```
LaunchpadApp (App Entry Point)
├── WindowAccessor (Window Configuration)
├── PagedGridView (Main Container)
├── SettingsView (Settings Overlay)
└── VisualEffectView (Background)
```

### 🧠 **Managers**

#### `AppManager` (Singleton)
- **Primary Role**: Application discovery, layout management, and persistence
- **Key Methods**:
  - `loadGridItems(appsPerPage:)`: Discovers apps and loads saved layout
  - `saveGridItems()`: Persists current layout to UserDefaults
  - `recalculatePages(appsPerPage:)`: Redistributes items across pages
  - `importLayout(appsPerPage:)` / `exportLayout()`: JSON-based layout backup
  - `clearGridItems(appsPerPage:)`: Resets to default alphabetical order

#### `SettingsManager` (Singleton)
- **Primary Role**: User preferences and configuration management
- **Settings Include**:
  - Grid dimensions (columns: 4-12, rows: 3-10)
  - Icon size (20-200px)
  - Animation delays and scroll thresholds
  - Folder layout configuration

### 📊 **Data Models**

#### `AppGridItem` (Enum)
```swift
enum AppGridItem: Identifiable, Equatable {
    case app(AppInfo)
    case folder(Folder)
}
```
- **Unified Interface**: Handles both applications and folders consistently
- **Serialization**: JSON export/import with `serialize()` method
- **Page Management**: Each item tracks its page location

#### `AppInfo` (Struct)
- **Properties**: `id`, `name`, `icon` (NSImage), `path`, `page`
- **Icon Processing**: Flattened for consistency using `flattenedForConsistency()`
- **Localization**: Uses `kMDItemDisplayName` for proper app names

#### `Folder` (Struct)
- **Properties**: `id`, `name`, `page`, `apps: [AppInfo]`
- **Preview**: Shows up to 9 apps in 3x3 grid preview
- **Dynamic Management**: Apps can be added/removed via drag-and-drop

#### `LaunchpadSettings` (Codable Struct)
- **Grid Configuration**: columns, rows, iconSize, dropDelay
- **Folder Settings**: folderColumns, folderRows
- **Interaction Settings**: scrollDebounceInterval, scrollActivationThreshold
- **Feature Flags**: showDock, transparency
- **Validation**: Enforces min/max bounds for all numeric settings

### 🎛️ **View Components**

#### Main Views
- **`PagedGridView`**: Primary container with horizontal page navigation
- **`SinglePageView`**: Individual page with LazyVGrid layout
- **`SearchResultsView`**: Vertical scrolling search results
- **`FolderDetailView`**: Modal folder content editor
- **`SettingsView`**: Tabbed settings interface (Layout + Features + Actions)

#### UI Components
- **`GridItemView`**: Unified renderer for apps and folders
- **`AppIconView`**: Application icon with name label
- **`FolderIconView`**: 3x3 grid preview of folder contents
- **`FolderNameView`**: Editable folder name header in folder detail view
- **`PageIndicatorView`**: Clickable page dots navigation
- **`SearchBarView`**: Top search input with glass morphism styling

#### Settings Components
- **`LayoutSettings`**: Grid dimensions and icon size configuration
- **`FeaturesSettings`**: Feature flags and interaction settings (dock, transparency, animations)
- **`ActionsSettings`**: Layout import/export and app management actions
- **`SettingsSlider`**: Reusable slider control with min/max labels
- **`SettingsNumberField`**: Numeric input with stepper control
- **`SettingsToggle`**: Toggle switch for boolean settings

#### Specialized Components
- **`DropZoneView`**: Left/right page navigation drop targets
- **`PageDropZonesView`**: Container for navigation drop zones
- **`EmptySearchView`**: No results state for search
- **`VisualEffectView`**: NSVisualEffectView wrapper for glass morphism

### 🎯 **Drag & Drop System**

#### Drop Delegates
- **`ItemDropDelegate`**: Handles app-to-app and app-to-folder drops, page overflow management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kristof12345/Launchpad](https://github.com/kristof12345/Launchpad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
