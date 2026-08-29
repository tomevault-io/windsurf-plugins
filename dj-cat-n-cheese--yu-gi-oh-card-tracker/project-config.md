---
trigger: always_on
description: > **Note to AI Agents:** This document is the authoritative source of truth for the codebase. If `README.md` contradicts this file regarding architecture or implementation details, trust this file.
---

# OpenYuGi - Architectural Manual

> **Note to AI Agents:** This document is the authoritative source of truth for the codebase. If `README.md` contradicts this file regarding architecture or implementation details, trust this file.

---

## 1. Core Philosophy & Constraints

*   **Local-First / No-Database**: We do not use SQL or NoSQL databases. The "Database" is a set of JSON files in `data/collections/`.
    *   *Constraint*: Operations must handle loading entire collections into memory (Pydantic models) and serializing them back.
    *   *Constraint*: Atomic writes are simulated by full file overwrites.
*   **NiceGUI Framework**: The UI is server-side rendered Python.
    *   *Pitfall*: **Blocking I/O kills the UI.** All file operations, network requests, or heavy computations (OCR) MUST run in `await run.io_bound(...)` or separate threads (ScannerManager).
    *   *Pitfall*: `ui.run_javascript` is asynchronous. Do not rely on its return value immediately in synchronous code blocks.
*   **Data Integrity**: The `CollectionEditor` class is the ONLY authorized way to mutate collection state. Do not modify `Collection.cards` lists directly in UI code.

---

## 2. Directory Structure

| Directory | Responsibility | Key Files |
| :--- | :--- | :--- |
| `src/core/` | **Domain Logic**. Pydantic models, Config, Persistence. **NO UI Code.** | `models.py`, `persistence.py`, `config_manager.py` |
| `src/services/` | **Integration**. External APIs, Hardware (Cam), Filesystem. | `ygo_api.py`, `image_manager.py`, `collection_editor.py` |
| `src/services/scanner/` | **Computer Vision**. Neural networks, OCR pipelines. | `manager.py`, `pipeline.py` |
| `src/ui/` | **Presentation**. NiceGUI pages and layout. | `scan.py`, `bulk_add.py`, `storage.py` |
| `src/ui/components/` | **Reusables**. Widgets used across multiple pages. | `single_card_view.py`, `filter_pane.py` |
| `data/` | **User Data**. The "Production DB". | `collections/*.json`, `images/*.jpg`, `db/card_db.json` |

---

## 3. Data Model Deep Dive (`src/core/models.py`)

### 3.1. Collection Hierarchy
The data structure is hierarchical to support aggregation and variant management.

1.  **`Collection`**: Root object. Contains list of `CollectionCard`.
    *   `storage_definitions`: List of defined Binders/Boxes.
2.  **`CollectionCard`**: Represents an abstract card identity (e.g., "Blue-Eyes").
    *   Key: `card_id` (Integer, from YGOPRODeck).
    *   Children: List of `CollectionVariant`.
3.  **`CollectionVariant`**: A specific printing (e.g., "LOB-EN001 Ultra Rare").
    *   Key: `variant_id` (Hash of card_id + set_code + rarity + image_id).
    *   *Critical*: Image ID is part of the key to support Alt Arts in same set.
4.  **`CollectionEntry`**: A physical stack of cards.
    *   Fields: `condition`, `language`, `first_edition`, `storage_location`, `quantity`.
    *   *Logic*: Entries are unique by (Cond + Lang + 1st + Storage). Adding a 2nd "Near Mint EN" card increments `quantity` of the existing entry, it does NOT create a new entry.

### 3.2. Reference Data (`ApiCard`)
*   `ApiCard` objects are transient (loaded from `card_db.json` cache).
*   They are **never** serialized into the user's collection JSON.
*   The UI joins `CollectionCard` (User Data) with `ApiCard` (Metadata) at runtime using `ygo_service`.

---

## 4. Core Services

### 4.1. Collection Editor (`src/services/collection_editor.py`)
The Transaction Manager.
*   **`apply_change(...)`**: The universal mutator.
    *   Handles "Add", "Remove", "Set Quantity".
    *   Automatically creates/cleans up parent `CollectionCard` and `CollectionVariant` objects if they become empty.
    *   *Key Logic*: If `variant_id` is not provided, it generates one deterministically using `generate_variant_id`.
*   **`move_card(...)`**: Atomic transfer between storage locations.
    *   Implemented as `remove(from_storage)` then `add(to_storage)`.
    *   Validates availability before execution.

### 4.2. Scanner Architecture (`src/services/scanner/`)
An Event-Driven State Machine.
*   **`ScannerManager` (Daemon)**:
    *   Runs in a separate `threading.Thread` to avoid blocking NiceGUI.
    *   Maintains a `scan_queue` of `ScanRequest` objects.
    *   Emits events (`scan_started`, `step_complete`, `scan_finished`) via `_emit`.
    *   UI listens via `register_listener` and updates via `ui.timer` consumer loop (avoiding direct thread-to-UI calls which crash NiceGUI).
*   **Pipeline Steps**:
    1.  **Preprocessing**: Edge Detection -> Perspective Warp.
    2.  **Dual-Track OCR**: Runs EasyOCR (Track 1) and DocTR (Track 2) in parallel on the warped image.
    3.  **Visual Analysis**: Detects Foil (Rarity) and "1st Edition" text.
    4.  **Art Matching**: Computes feature vector (YOLOv8 Classify) and compares against `data/images` index.
    5.  **Heuristic Matching**: Scores candidates based on Set Code (80pts), Name (50pts), and Art (40pts).

### 4.3. Image Manager (`src/services/image_manager.py`)
*   **Lazy Loading**: Images are not downloaded until requested by the UI.
*   **Resolutions**:
    *   `{id}.jpg`: Standard resolution.
    *   `{id}_high.jpg`: High-Res.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DJ-Cat-N-Cheese/Yu-Gi-Oh-Card-Tracker](https://github.com/DJ-Cat-N-Cheese/Yu-Gi-Oh-Card-Tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
