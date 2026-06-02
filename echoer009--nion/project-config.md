---
trigger: always_on
description: Nion is an AI companion + task management Android app. Rust core library exposed to Kotlin via UniFFI. Axum backend exists but is not yet wired to the app.
---

# AGENTS.md

## Project Overview

Nion is an AI companion + task management Android app. Rust core library exposed to Kotlin via UniFFI. Axum backend exists but is not yet wired to the app.

## Workspace Layout

- **`core/`** — Rust library (`nion-core`). SQLite-backed task/checklist CRUD, settings. Exposed as `cdylib` + `lib`.
- **`backend/`** — Axum HTTP server (`nion-backend`). Stub; depends on `nion-core`.
- **`app/`** — Android app (Kotlin, Jetpack Compose, Material 3). Package: `com.echonion.nion`.
- **`tools/uniffi-bindgen/`** — CLI wrapper for UniFFI binding generation (`uniffi-bindgen-cli`).
- **`docs/`** — Design notes, pitfalls, audit reports.

## Commands

### Rust

```bash
cargo build -p nion-core              # build core library
cargo test -p nion-core               # run core tests (inline #[cfg(test)] mod in nion_core.rs)
cargo run -p nion-backend             # start backend on :3000
cargo run -p uniffi-bindgen-cli -- generate --library <path-to-so> --language kotlin --out-dir <out>
```

### Android

```bash
cd app && ./gradlew assembleDebug     # build APK
./build-android.sh                    # cross-compile Rust for Android + generate UniFFI Kotlin bindings + copy artifacts
./deploy.sh                           # build APK, install via adb, launch app (WSL + usbipd)
```

## Build & Deploy Flow

1. `build-android.sh` compiles `nion-core` for `aarch64-linux-android` and `x86_64-linux-android` using NDK 27.
2. Runs `uniffi-bindgen` against the aarch64 `.so` to produce `nion_core.kt`.
3. Copies `.so` files to `app/app/src/main/jniLibs/{arm64-v8a,x86_64}/` and `.kt` binding to `app/app/src/main/java/uniffi/nion_core/`.
4. `deploy.sh` runs Gradle build, installs via `adb`, launches `com.echonion.nion/.MainActivity`.

## UniFFI Conventions

- **Proc-macro mode only** — no UDL files, no `build.rs`, no `uniffi` build feature.
- `uniffi::setup_scaffolding!()` must be called at the top of `core/src/lib.rs`.
- Types use `#[uniffi::Record]`, `#[uniffi::Object]`, `#[uniffi::Error]`, `#[uniffi::export]`.
- Do not mix `setup_scaffolding!()` with `include_scaffolding!("xxx.udl")`.

## Architecture Notes

- `NionCore` holds `db: Mutex<Connection>` — single Rust instance, but Kotlin side historically created two (see audit C1). The `core()` extension on `Application` is the intended singleton.
- All DB ops are synchronous on the caller thread. Kotlin callers should use `Dispatchers.IO`.
- `reorder_tasks` uses manual `BEGIN`/`COMMIT`; other multi-step DB ops do not.
- The app uses a `DualPanelLayout` with left sidebar (checklists) and right sidebar (companion), both swipeable.
- Navigation: `tasks`, `schedule`, `pomodoro`, `settings` routes via Jetpack Navigation Compose.
- **分组 (Groups)**: 清单下的二级分类。数据模型: `Checklist → Group → Task`。`TaskData.group_id` 关联分组，`GroupData.checklist_id` 关联清单。

## Data Model

```
Checklist (清单) ──1:N──> Group (分组) ──1:N──> Task (任务)
Checklist (清单) ──1:N──> Task (任务, via category_id, group_id 可为空)
Task (任务) ──1:N──> Task (子任务, via parent_id)
```

## Backend API (Axum on :3000)

### 清单 API
- `GET    /api/checklists` — 获取所有清单
- `POST   /api/checklists` — 创建清单 `{ "name": "..." }`
- `PUT    /api/checklists/:id` — 修改清单名称 `{ "name": "..." }`
- `DELETE /api/checklists/:id` — 删除清单
- `PUT    /api/checklists/reorder` — 重排清单 `{ "ordered_ids": [...] }`

### 分组 API
- `POST   /api/groups` — 创建分组 `{ "name": "语文", "checklist_id": "...", "color": "#FF5722" }`
- `GET    /api/groups/:checklist_id` — 获取清单下的所有分组
- `PUT    /api/groups/:id` — 更新分组 `{ "name": "...", "color": "..." }`
- `DELETE /api/groups/:id` — 删除分组（保留组内任务，group_id 置空）
- `PUT    /api/groups/reorder` — 重排分组 `{ "ordered_ids": [...] }`

### 任务 API
- `GET    /api/tasks` — 获取所有任务
- `GET    /api/tasks/by-category?category_id=...&group_id=...` — 按清单和分组筛选
- `GET    /api/tasks/:id` — 获取单个任务
- `POST   /api/tasks` — 创建任务 `{ "title": "...", "group_id": "...", ... }`
- `PUT    /api/tasks/:id` — 更新任务（支持 `group_id` 字段）
- `DELETE /api/tasks/:id` — 删除任务
- `GET    /api/tasks/:id/subtasks` — 获取子任务
- `PUT    /api/tasks/:id/parent` — 更新父任务 `{ "new_parent_id": "..." }`
- `PUT    /api/tasks/:id/group` — 更新分组归属 `{ "group_id": "..." }`
- `POST   /api/tasks/:id/focus` — 累加专注时长 `{ "seconds": 60 }`
- `PUT    /api/tasks/reorder` — 重排任务 `{ "ordered_ids": [...] }`

### 设置 API
- `GET    /api/settings/:key` — 获取设置值
- `PUT    /api/settings` — 设置键值对 `{ "key": "...", "value": "..." }`

## Key Files

| What | Path |
|------|------|
| Rust core API | `core/src/nion_core.rs` |
| Rust models/errors | `core/src/models.rs` |
| Kotlin entry composable | `app/app/src/main/java/com/echonion/nion/ui/NionApp.kt` |
| Task screen + ViewModel | `app/app/src/main/java/com/echonion/nion/ui/task/` |
| Memory tool | `app/app/src/main/java/com/echonion/nion/ui/companion/tools/MemoryTool.kt` |
| UniFFI binding output | `app/app/src/main/java/uniffi/nion_core/nion_core.kt` |
| Known pitfalls | `docs/pitfalls.md` |
| Code audit | `docs/code-audit-report.md` |

## Coding Rules

### 不要主动修改不是本次任务范围的代码

如果某个文件存在编译错误或代码问题，但该文件**不是你当前任务涉及的目标文件**，不要主动去修复它。只在用户明确要求时才修改。

### 禁止卸载应用

**绝对不能执行 `adb uninstall` 或任何卸载操作**，必须先询问用户确认。卸载会清除所有本地数据（任务、对话、设置等），造成不可逆的数据丢失。

### 注释规范

所有新增和修改的代码**必须写详细注释**。要求：


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Echoer009/Nion](https://github.com/Echoer009/Nion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
