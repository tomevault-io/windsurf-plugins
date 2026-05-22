---
trigger: always_on
description: State Manager Roles and Boundaries (Routing Guide)
---


Use this to decide which manager to call, what each owns, and where its jurisdiction ends. Keep manager docstrings implementation‑agnostic; this guide is only for high‑level routing and composition.

### ConversationManager
- **Role**: Live chat orchestrator. Routes user requests to `Actor` for code-first execution and wires steering (pause/resume/interject/stop) during conversations.
- **Scope**: Conversation‑level control and message flow; returns/relays steerable handles from inner tools.
- **Connections**:
  - **Steered by**: Top-level UI/controller (outside managers).
  - **Steers**: `Actor.act` (central intelligence); relays in‑flight handles from `Actor.act` and `TaskScheduler.execute`.

### Actor
- **Role**: Central intelligence that orchestrates all state managers through code-first plans. Generates and executes Python plans that call primitives directly.
- **Scope**: Code-first execution via `act()` method. Generates Python plans that orchestrate `primitives.contacts.*`, `primitives.knowledge.*`, `primitives.tasks.*`, etc. Wires in‑flight handles to `ConversationManager` for real‑time steering.
- **Connections**:
  - **Steered by**: `ConversationManager` (primary caller of `act()`).
  - **Steers**: All state manager primitives (`primitives.contacts.*`, `primitives.knowledge.*`, `primitives.tasks.*`, etc.), `TaskScheduler`, and the `ConversationManager` handle (`ask`/`interject`/`get_full_transcript`). Uses `FunctionManager` for function discovery and execution.

### Actor routing playbook
- **Read‑only questions**
  - Tasks → `primitives.tasks.ask` or `TaskScheduler.ask`
  - Contacts → `primitives.contacts.ask`
  - Transcripts → `primitives.transcripts.ask` (may call `primitives.contacts.ask` for participants)
  - Knowledge → `primitives.knowledge.ask`
  - Secrets (metadata/placeholders only) → `primitives.secrets.ask`
  - Time‑sensitive/web ("today/latest/now") → `primitives.web.ask`
  - About a specific received file (filename known) → `primitives.files.ask`
- **Mutations (create/edit/delete/merge)**
  - Tasks/queues/ordering → `primitives.tasks.update` or `TaskScheduler.update`
  - Contacts → `primitives.contacts.update`
  - Knowledge/schema changes or ingestion → `primitives.knowledge.update` / `primitives.knowledge.refactor`
  - Guidance → `GuidanceManager_add_guidance` / `GuidanceManager_update_guidance` / `GuidanceManager_delete_guidance` (top-level JSON tool calls)
  - Secrets → `primitives.secrets.update`
- **Execution vs. interaction**
  - Ephemeral live action (UI control/one‑off interaction) → `Actor.act` (called via ConversationManager)
  - Durable, tracked work → `TaskScheduler.execute` (via `primitives.tasks.execute`)
  - Never use `TaskScheduler.update` to start work; always use `execute`.
- **File ingestion pipeline**
  - `primitives.files.parse` → `primitives.knowledge.update` (to persist structured facts)
- **Images**
  - Images are referenced **by filesystem path** across the entire stack. Screenshot directories (`Screenshots/User/`, `Screenshots/Assistant/`) and other workspace paths serve as the universal cross‑manager pointer for visual content. Managers and plans reference images via their relative filepath — no special `images` parameter or structured ref types are needed at the public API boundary.

### ImageManager
- **Role**: Persistent image store and metadata registry. Provides durable `image_id`‑keyed storage in the `Images` context, backing filesystem images with cloud persistence and queryable metadata.
- **Data model & identity**:
  - Every stored image has a unique numeric `image_id` (stable within the active assistant context).
  - Image rows store base64 bytes or a cloud object URL, plus metadata (caption, timestamp, mime/type).
  - Each image row carries an optional `filepath` field that records the local filesystem path the image was saved to. This is the bridge between the filesystem‑based reference convention and the persistent `Images` context.
- **ImageHandle wrapper**:
  - Internal code operates on an `ImageHandle` abstraction that wraps an image row and exposes:
    - `image_id`, optional `caption`/metadata, `filepath`
    - `raw()` → returns image bytes (resolves base64 vs signed URL transparently)
    - `ask(question)` → sends the image to a vision‑capable model and returns a text answer
- **Cross‑manager image convention**:
  - **Filesystem paths are the universal image reference.** When images need to flow between managers (e.g., from `Actor` plans into `GuidanceManager.update`), they are referenced by their workspace‑relative filepath (e.g., `Screenshots/User/2026-02-16T14-30-45.jpg`). The receiving manager or its internal tools can resolve filepaths to `image_id`s via `ImageManager.filter_images(filter="filepath == '...'")` when persistent storage linkage is needed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unifyai/unity](https://github.com/unifyai/unity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
