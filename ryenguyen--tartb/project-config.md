---
trigger: always_on
description: Provides the glass-morphism container with:
---

# TArtB - Art New Tab Chrome Extension

A Chrome extension that replaces your new tab page with stunning artwork from world-renowned museums.

## Tech Stack

- **Framework:** React 19 + TypeScript + Vite 7
- **State:** Zustand 5 (with Chrome storage persistence)
- **Data Fetching:** TanStack React Query 5
- **Backend:** Firebase Firestore + Cloud Functions
- **Styling:** Tailwind CSS 4 + Framer Motion
- **Date/Calendar:** date-fns + react-day-picker v9
- **Popover/Positioning:** @floating-ui/react
- **Drag & Drop:** @dnd-kit/core + @dnd-kit/sortable
- **i18n:** i18next (EN, VI)
- **Extension:** Manifest v3, new tab override

## Project Structure

```
src/
├── App.tsx                    # Main app, language sync, dock + sidebar wiring
├── main.tsx                   # React Query setup, i18n import
├── i18n.ts                    # i18next config
├── components/
│   ├── Artwork/
│   │   └── InteractiveArtwork.tsx  # Canvas-based image with dissolve effect
│   ├── icons/                 # Grip (drag handle icon)
│   ├── atoms/                 # GlassButton, Glass, Switch, Selector, Clock, Typography, WidgetWrapper, Popover, DatePicker, Dropdown, Toast
│   ├── molecules/             # ArtworkInfo, MenuCategories, DockStation, ToDo (with drag-drop components)
│   └── organisms/             # Sidebar (settings panel), DynamicFieldRenderer
├── assets/
│   └── icons/                 # SVG icons (Close, CheckCircle, Error, Info, Flag, Calendar, etc.)
├── stores/
│   ├── artworkStore.ts        # Current artwork state
│   ├── settingsStore.ts       # Widget + app settings with Chrome storage middleware
│   ├── todoStore.ts           # Task lists & tasks with Chrome storage persistence & error handling
│   └── toastStore.ts          # Toast notifications with auto-dismiss
├── services/
│   ├── api/
│   │   ├── artInstituteApi.ts # Chicago Art Institute API
│   │   ├── metMuseumApi.ts    # Metropolitan Museum API
│   │   └── wikiArtApi.ts      # WikiArt API (needs auth)
│   ├── firebase/
│   │   └── firestoreService.ts # Primary data source
│   └── todo/
│       └── todoService.ts      # Hybrid service: LocalTodoService + FirestoreTodoService with granular operations
├── hooks/
│   ├── useArtwork.ts          # Main hook: Firestore → API fallback chain
│   └── useToDo.ts             # ToDo state: input, priority, deadline, task CRUD, drag-drop reordering
├── types/
│   ├── settings.ts            # WidgetId enum, UserSettings, widget state interfaces
│   └── toDo.ts                # Task, TaskList, Tag, TaskGroup, TodoData interfaces
├── utils/
│   ├── stringUtils.ts         # generateId() - UUID generation with crypto API fallback
│   ├── objectUtils.ts         # removeUndefined() - clean objects before Firestore writes
│   ├── taskSortUtils.ts       # Task sorting logic (DATE, DUE_DATE, PRIORITY, TITLE, MANUAL)
│   ├── taskGroupUtils.ts      # Task grouping logic (PRIORITY, DATE, DUE_DATE, NONE)
│   └── taskReorderUtils.ts    # Drag-drop order calculation, cross-group property updates
├── constants/
│   ├── common.ts              # Enums: TimeFormat, ClockType, Language, FieldType, TaskPriorityType, TaskSortBy, TaskGroupBy
│   ├── widgets.ts             # WIDGET_REGISTRY - widget metadata (icons, names, categories)
│   ├── toDoConfig.ts          # PRIORITY_CONFIG - priority colors/labels
│   └── settingConfig.ts       # Settings UI structure by category
├── locales/                   # en.json, vi.json
└── animations/                # Framer Motion variants
```

## Key Patterns

### Widget System (macOS-style)
Each widget has two states:
- **enabled:** Widget appears in dock (red dot disables → removes from dock)
- **visible:** Widget appears on screen (yellow dot minimizes → stays in dock, dimmed)

```typescript
// Widget IDs
enum WidgetId {
  CLOCK = "clock",
  DATE = "date",
  ARTWORK_INFO = "artworkInfo",
  TODO = "toDo",
}

// Settings structure
interface UserSettings {
  widgets: {
    clock: { enabled: boolean, visible: boolean, type: ClockType, timeFormat: TimeFormat },
    date: { enabled: boolean, visible: boolean },
    artworkInfo: { enabled: boolean, visible: boolean },
  },
  artwork: { museum: string, changeInterval: number },
  app: { language: Language },
}
```

### DockStation Behavior
- Shows icons for **enabled** widgets only
- Dimmed icons for **minimized** widgets (enabled but not visible)
- **Click:** Toggle `visible` state
- **Long press (400ms):** Open settings for that widget's category

### WidgetWrapper
Provides the glass-morphism container with:
- 3D tilt effect on hover
- Yellow dot (minimize) and red dot (close) buttons
- Can connect directly to store via `widgetId` prop

### Data Fetching (useArtwork hook)
```
Firestore → Selected Museum API → Random Museum → Met Museum (fallback)
```
- React Query handles caching (based on changeInterval)
- Auto-refetch based on `settings.artwork.changeInterval`
- Progressive image loading (small → full)

### Todo Service Layer (Optimized)

**Architecture:** Hybrid service switches between LocalTodoService (anonymous users) and FirestoreTodoService (authenticated users)

**Service Interface:**
```typescript
interface TodoService {
  // Bulk operations (for initial load, migration, bulk deletes)
  load(), saveLists(), saveTasks(), saveTags(), clear()


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RyeNguyen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
