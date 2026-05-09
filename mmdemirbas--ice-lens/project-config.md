---
trigger: always_on
description: **Iceberg Lens** is a read-only desktop application for inspecting Apache Iceberg and Apache Paimon table structure from local filesystems. It renders an interactive graph visualization (table → metadata → snapshots → manifests → data files → sample rows) alongside a detailed inspector panel.
---

# CLAUDE.md

## Project overview

**Iceberg Lens** is a read-only desktop application for inspecting Apache Iceberg and Apache Paimon table structure from local filesystems. It renders an interactive graph visualization (table → metadata → snapshots → manifests → data files → sample rows) alongside a detailed inspector panel.

## Tech stack

- **Kotlin 2.3.10** + **JetBrains Compose Desktop 1.10.1** + **Material3**
- **Apache Avro 1.12.1** / **Avro4k 2.10.0** — manifest list & manifest file deserialization
- **DuckDB JDBC 1.4.4.0** — Parquet/ORC/Avro sample row queries
- **Eclipse ELK 0.11.0** — layered graph layout engine
- **Kotlinx Serialization 1.10.0** — JSON metadata parsing
- **Gradle 9.0.0** with Kotlin DSL; requires **Java 17+**

## Architecture

```
src/main/kotlin/
├── Main.kt                    # Entry point, window state persistence (multi-monitor aware)
├── model/
│   ├── IcebergSchema.kt       # @Serializable Iceberg data classes (metadata, snapshot, manifest, data file)
│   ├── IcebergPaths.kt        # Shared path utilities (normalizeFilePath, metadataVersionFromFileName)
│   ├── UnifiedModel.kt        # Aggregated data layer — reads & links all Iceberg artifacts into a tree
│   ├── PaimonSchema.kt        # @Serializable Paimon data classes (snapshot, schema, manifest list, manifest entry)
│   ├── PaimonUnifiedModel.kt  # Aggregated Paimon data layer — reads & links snapshots, schemas, manifests
│   ├── GraphTypes.kt          # GraphModel (with nodeById), GraphNode (sealed incl. Paimon types), GraphEdge
│   ├── WorkspaceTypes.kt      # WorkspaceItem sealed class (Warehouse / SingleTable), serialization
│   └── ToolWindowTypes.kt     # ToolWindowAnchor enum, ToolWindowConfig
├── service/
│   ├── AvroReader.kt          # Shared Avro file reader (reified readAvro<T>), used by both Iceberg and Paimon
│   ├── IcebergReader.kt       # Iceberg JSON/Avro reading (delegates Avro to AvroReader)
│   ├── PaimonReader.kt        # Paimon JSON snapshot/schema + Avro manifest list/manifest reading
│   ├── SampleRowReader.kt     # DuckDB JDBC queries for sample rows (Parquet, ORC, Avro — max 50)
│   ├── IcebergGraphBuilder.kt # Iceberg-specific graph construction: UnifiedTableModel → nodes + edges
│   ├── PaimonGraphBuilder.kt  # Paimon-specific graph construction: PaimonUnifiedTableModel → nodes + edges
│   ├── GraphLayoutService.kt  # Format-agnostic ELK layout + post-processing (ordering, alignment, overlap prevention)
│   └── TableFormatDetector.kt # Directory-based table format detection (Iceberg / Paimon / Unknown)
└── ui/
    ├── AppState.kt            # Business logic: workspace mgmt, table loading, caching, snapshot filter (testable, no UI)
    ├── App.kt                 # Thin UI layer — layout, keyboard shortcuts, LaunchedEffects (delegates to AppState)
    ├── AboutDialog.kt         # About dialog with version info, diagnostics, cheat sheet
    ├── Theme.kt               # Color schemes, dark surface detection, selection highlight
    ├── CommonComponents.kt    # Reusable widgets: draggable dividers, toolbar group/icon button
    ├── FormatUtils.kt         # Timestamp formatting, long set serialization
    ├── WorkspaceUtils.kt      # Table detection, recursive scanning, native file chooser, workspace dedup
    ├── SnapshotFilter.kt      # Snapshot filter data model and graph filtering logic
    ├── GraphCanvas.kt         # Interactive graph: zoom/pan, node selection/drag, marquee, mini-map, viewport culling
    ├── NodeComponents.kt      # Node card composables (Iceberg + Paimon node types) + tooltip + copy buttons
    ├── NodeDetails.kt         # Inspector panel — detailed metadata, JSON highlighting, changelogs, sample rows
    ├── Sidebar.kt             # Workspace panel — add/remove roots, search, drag-to-reorder, format badges (ICE/PMN)
    ├── NavigationTree.kt      # Structure tree view — flatten graph, search, expand/collapse
    └── ToolWindow.kt          # Draggable tool window bars and panes
```

## Build & run

```bash
./gradlew run          # Run the application
./gradlew build        # Build
./gradlew packageDmg   # macOS installer
./gradlew packageMsi   # Windows installer
./gradlew packageDeb   # Linux installer
```

## Key conventions

- State is persisted via `java.util.prefs.Preferences` under `com.github.mmdemirbas.icelens`
- All data access is read-only — no table modifications
- Node colors are hardcoded per node type in `NodeComponents.kt` (`getGraphNodeColor` / `getGraphNodeBorderColor`)
- Dark mode detection uses `perceivedBrightness()` (0.2126R + 0.7152G + 0.0722B < 0.5)
- Graph layout flow: `FormatTableModel` → `GraphLayoutService.layoutGraph()` dispatches to format-specific builder → `GraphBuildResult` → `layoutNodes()` → `GraphModel` → `GraphCanvas`
- `GraphModel.nodeById` provides a lazy `Map<String, GraphNode>` — use it instead of `nodes.find`/`nodes.associateBy`
- File paths are resolved relative to the metadata directory using `resolveForceRelative()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mmdemirbas/ice-lens](https://github.com/mmdemirbas/ice-lens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
