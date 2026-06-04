---
trigger: always_on
description: REST API design principles for mock endpoints
---


# REST API Design Principles

This rule defines REST API design patterns and best practices for the mocks server handlers.

## CRUD Endpoint Patterns

### GET Operations (Read)

#### Get All Resources
```rust
// GET /{resource} - List all items in a resource
pub async fn get_all(
    State(state): State<SharedState>,
    Path(resource): Path<String>,
    Query(params): Query<HashMap<String, String>>,
) -> Result<Json<Value>, StatusCode> {
    let state = state.lock().await;
    
    // Support filtering with query parameters
    let data = if params.is_empty() {
        select_all(&state.storage.data, &resource)
    } else {
        select_with_filter(&state.storage.data, &resource, &params)
    };
    
    match data {
        Ok(result) => Ok(Json(result)),
        Err(_) => Err(StatusCode::NOT_FOUND),
    }
}
```

#### Get Single Resource
```rust
// GET /{resource}/{id} - Get specific item by ID
pub async fn get_one(
    State(state): State<SharedState>,
    Path((resource, id)): Path<(String, String)>,
) -> Result<Json<Value>, StatusCode> {
    let state = state.lock().await;
    
    match select_one(&state.storage.data, &resource, &id) {
        Ok(item) => Ok(Json(item)),
        Err(_) => Err(StatusCode::NOT_FOUND),
    }
}
```

### POST Operations (Create)

#### Create New Resource
```rust
// POST /{resource} - Create new item
pub async fn post(
    State(state): State<SharedState>,
    Path(resource): Path<String>,
    Json(input): Json<Value>,
) -> Result<(StatusCode, Json<Value>), StatusCode> {
    let mut state = state.lock().await;
    
    // Validate input before processing
    if let Err(_) = validate_input(&input) {
        return Err(StatusCode::BAD_REQUEST);
    }
    
    match insert(&mut state.storage.data, &resource, input) {
        Ok(created_item) => {
            // Write to file if overwrite enabled
            if state.storage.overwrite {
                if let Err(_) = Writer::new(&state.storage.file)
                    .write(&state.storage.data) {
                    return Err(StatusCode::INTERNAL_SERVER_ERROR);
                }
            }
            Ok((StatusCode::CREATED, Json(created_item)))
        }
        Err(_) => Err(StatusCode::BAD_REQUEST),
    }
}
```

### PUT Operations (Replace)

#### Replace Entire Resource
```rust
// PUT /{resource}/{id} - Replace entire item
pub async fn put(
    State(state): State<SharedState>,
    Path((resource, id)): Path<(String, String)>,
    Json(input): Json<Value>,
) -> Result<Json<Value>, StatusCode> {
    let mut state = state.lock().await;
    
    match replace(&mut state.storage.data, &resource, &id, input) {
        Ok(updated_item) => {
            if state.storage.overwrite {
                let _ = Writer::new(&state.storage.file)
                    .write(&state.storage.data);
            }
            Ok(Json(updated_item))
        }
        Err(_) => Err(StatusCode::NOT_FOUND),
    }
}

// PUT /{resource} - Replace entire collection (special case)
pub async fn put_one(
    State(state): State<SharedState>,
    Path(resource): Path<String>,
    Json(input): Json<Value>,
) -> Result<Json<Value>, StatusCode> {
    let mut state = state.lock().await;
    
    match replace_one(&mut state.storage.data, &resource, input) {
        Ok(result) => {
            if state.storage.overwrite {
                let _ = Writer::new(&state.storage.file)
                    .write(&state.storage.data);
            }
            Ok(Json(result))
        }
        Err(_) => Err(StatusCode::NOT_FOUND),
    }
}
```

### PATCH Operations (Partial Update)

#### Partial Update of Resource
```rust
// PATCH /{resource}/{id} - Partial update of item
pub async fn patch(
    State(state): State<SharedState>,
    Path((resource, id)): Path<(String, String)>,
    Json(input): Json<Value>,
) -> Result<Json<Value>, StatusCode> {
    let mut state = state.lock().await;
    
    match update(&mut state.storage.data, &resource, &id, input) {
        Ok(updated_item) => {
            if state.storage.overwrite {
                let _ = Writer::new(&state.storage.file)
                    .write(&state.storage.data);
            }
            Ok(Json(updated_item))
        }
        Err(_) => Err(StatusCode::NOT_FOUND),
    }
}

// PATCH /{resource} - Partial update of collection
pub async fn patch_one(
    State(state): State<SharedState>,
    Path(resource): Path<String>,
    Json(input): Json<Value>,
) -> Result<Json<Value>, StatusCode> {
    let mut state = state.lock().await;
    
    match update_one(&mut state.storage.data, &resource, input) {
        Ok(result) => {
            if state.storage.overwrite {
                let _ = Writer::new(&state.storage.file)
                    .write(&state.storage.data);
            }
            Ok(Json(result))
        }
        Err(_) => Err(StatusCode::NOT_FOUND),
    }
}
```

### DELETE Operations (Remove)

#### Delete Resource
```rust
// DELETE /{resource}/{id} - Delete specific item
pub async fn delete(
    State(state): State<SharedState>,
    Path((resource, id)): Path<(String, String)>,
) -> Result<StatusCode, StatusCode> {
    let mut state = state.lock().await;
    
    match remove(&mut state.storage.data, &resource, &id) {
        Ok(_) => {
            if state.storage.overwrite {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mocks-rs/mocks](https://github.com/mocks-rs/mocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
