---
trigger: always_on
description: Architecture patterns specific to mocks CLI tool
---


# Mocks Architecture Patterns

This rule defines architecture-specific patterns for the mocks CLI tool, focusing on dynamic routing, shared state management, and storage abstraction.

## Dynamic Routing Implementation

### JSON Structure to HTTP Endpoints
The core feature of mocks is converting JSON file structure into REST API endpoints automatically.

```rust
// Convert JSON keys to resource paths
fn convert_to_resource_paths(value: &Value) -> Vec<String> {
    let mut paths = vec![];
    
    if let Value::Object(obj) = value {
        for (key, _) in obj {
            // Handle nested paths like "api/v1/users" -> "/api/v1/{resource}"
            if let Some(last_slash) = key.rfind('/') {
                let (prefix, _) = key.split_at(last_slash + 1);
                paths.push(format!("/{prefix}{{resource}}"));
            } else {
                paths.push("/{resource}".to_string());
            }
        }
    }
    
    // Sort by depth (deeper paths first)
    paths.sort_by(|a, b| {
        let a_count = a.matches('/').count();
        let b_count = b.matches('/').count();
        b_count.cmp(&a_count)
    });
    
    paths
}
```

### Router Creation with Dynamic Paths
```rust
fn create_router(state: SharedState, value: &Value) -> Router {
    // Base routers for health check and CRUD operations
    let hc_router = Router::new().route("/", get(hc));
    let storage_router = Router::new()
        .route("/", get(get_all).post(post).put(put_one).patch(patch_one))
        .route("/{id}", get(get_one).put(put).patch(patch).delete(delete));

    let mut router = Router::new().nest("/_hc", hc_router);

    // Add dynamic resource paths
    let resource_paths = convert_to_resource_paths(value);
    for path in resource_paths {
        router = router.nest(&path, storage_router.clone().with_state(state.clone()));
    }

    router
}
```

### Resource Discovery Pattern
```rust
impl Storage {
    /// Get all available resource names from storage data
    pub fn resources(&self) -> Vec<String> {
        if let Value::Object(obj) = &self.data {
            obj.keys().cloned().collect()
        } else {
            vec![]
        }
    }
    
    /// Check if a resource exists in storage
    pub fn has_resource(&self, resource: &str) -> bool {
        if let Value::Object(obj) = &self.data {
            obj.contains_key(resource)
        } else {
            false
        }
    }
}
```

## Shared State Management

### AppState Design Pattern
```rust
// Define shared application state
pub struct AppState {
    pub storage: Storage,
}

pub type SharedState = Arc<Mutex<AppState>>;

impl AppState {
    pub fn new(storage: Storage) -> SharedState {
        Arc::new(Mutex::new(AppState { storage }))
    }
}
```

### Thread-Safe State Access
```rust
// Pattern for accessing shared state in handlers
pub async fn get_all(
    State(state): State<SharedState>,
    Path(resource): Path<String>,
) -> Result<Json<Value>, StatusCode> {
    let state = state.lock().await;
    match select_all(&state.storage.data, &resource) {
        Ok(data) => Ok(Json(data)),
        Err(_) => Err(StatusCode::NOT_FOUND),
    }
}
```

### State Mutation with Atomic Operations
```rust
pub async fn post(
    State(state): State<SharedState>,
    Path(resource): Path<String>,
    Json(input): Json<Value>,
) -> Result<Json<Value>, StatusCode> {
    let mut state = state.lock().await;
    
    // Perform atomic storage operation
    match insert(&mut state.storage.data, &resource, input) {
        Ok(result) => {
            // Write back to file if overwrite is enabled
            if state.storage.overwrite {
                if let Err(_) = Writer::new(&state.storage.file)
                    .write(&state.storage.data) {
                    return Err(StatusCode::INTERNAL_SERVER_ERROR);
                }
            }
            Ok(Json(result))
        }
        Err(_) => Err(StatusCode::BAD_REQUEST),
    }
}
```

## Storage Abstraction Layer

### Storage Interface Pattern
```rust
impl Storage {
    /// Create a new Storage instance with validation
    pub fn new(path: &str, overwrite: bool) -> Result<Storage, MocksError> {
        let data = Reader::new(path).read()?;
        
        // Validate storage structure
        Self::validate_storage_structure(&data)?;
        
        Ok(Storage {
            file: path.to_string(),
            data,
            overwrite,
        })
    }
    
    fn validate_storage_structure(data: &Value) -> Result<(), MocksError> {
        match data {
            Value::Object(_) => Ok(()),
            _ => Err(MocksError::InvalidJson(
                "Storage must be a JSON object".to_string()
            )),
        }
    }
}
```

### Reader/Writer Abstraction
```rust
// File reading abstraction
pub struct Reader {
    path: String,
}

impl Reader {
    pub fn new(path: &str) -> Self {
        Self {
            path: path.to_string(),
        }
    }
    
    pub fn read(&self) -> Result<Value, MocksError> {
        let content = fs::read_to_string(&self.path)
            .map_err(|e| MocksError::FailedReadFile(e.to_string()))?;
        
        serde_json::from_str(&content)
            .map_err(|e| MocksError::InvalidJson(e.to_string()))
    }
}

// File writing abstraction with atomic operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mocks-rs/mocks](https://github.com/mocks-rs/mocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
