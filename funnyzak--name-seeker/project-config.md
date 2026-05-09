---
trigger: always_on
description: Rust and Tauri Backend Development Rules
---


# Rust + Tauri Backend Development Rules

## 🦀 Rust Code Standards

### Error Handling
- Use `Result<T, E>` to handle operations that may fail
- Define custom error types implementing `Display` and `Error` traits
- Use `?` operator for error propagation

```rust
// ✅ Correct error handling
#[derive(Debug, thiserror::Error)]
pub enum SearchError {
    #[error("Network request failed: {0}")]
    NetworkError(#[from] reqwest::Error),
    #[error("Parse error: {0}")]
    ParseError(String),
}

pub async fn search_username(query: &str) -> Result<Vec<SearchResult>, SearchError> {
    let response = reqwest::get(&url).await?;
    let results: Vec<SearchResult> = response.json().await?;
    Ok(results)
}
```

### Async Programming
- Use `async/await` syntax
- Use `tokio::spawn` appropriately for concurrency
- Use `Arc<Mutex<T>>` to handle shared state

```rust
// ✅ Correct async programming
pub struct SearchEngine {
    config: SearchConfig,
    client: reqwest::Client,
}

impl SearchEngine {
    pub async fn search(&self, sites: &[Site]) -> Result<Vec<SearchResult>, SearchError> {
        let semaphore = Arc::new(Semaphore::new(self.config.max_concurrent_requests));
        let mut handles = Vec::new();
        
        for site in sites {
            let semaphore = semaphore.clone();
            let client = self.client.clone();
            
            let handle = tokio::spawn(async move {
                let _permit = semaphore.acquire().await?;
                self.check_site(&client, site).await
            });
            
            handles.push(handle);
        }
        
        // Wait for all tasks to complete
        let mut results = Vec::new();
        for handle in handles {
            if let Ok(result) = handle.await? {
                results.push(result);
            }
        }
        
        Ok(results)
    }
}
```

## 🎯 Tauri Command Development

### Command Function Standards
- Use `#[tauri::command]` macro
- Use snake_case for function names
- Return `Result<T, String>` type

```rust
// ✅ Correct Tauri commands
#[tauri::command]
async fn start_search(
    app: AppHandle,
    query: String,
    search_type: Option<String>,
    state: tauri::State<'_, AppState>,
) -> Result<(), String> {
    // Validate input
    validate_input(&query)?;
    
    // Execute search
    let search_engine = SearchEngine::new(config)?;
    let results = search_engine.search(&sites).await
        .map_err(|e| e.to_string())?;
    
    // Send event to frontend
    app.emit("search-update", &results)
        .map_err(|e| e.to_string())?;
    
    Ok(())
}
```

### State Management
- Use `tauri::State` to manage application state
- Use `Arc<Mutex<T>>` to handle concurrent access
- Use lifetimes appropriately

```rust
// ✅ Correct state management
#[derive(Default)]
pub struct AppState {
    pub search_handle: Arc<Mutex<Option<JoinHandle<()>>>>,
    pub search_results: Arc<Mutex<Vec<SearchResult>>>,
}

#[tauri::command]
async fn get_search_results(
    state: tauri::State<'_, AppState>
) -> Result<Vec<SearchResult>, String> {
    let results = state.search_results.lock().await;
    Ok(results.clone())
}
```

## 🔧 Module Organization

### Module Structure
- Use `mod.rs` files to organize modules
- Each module has clear responsibilities
- Use `pub` to control visibility

```rust
// src-tauri/src/core/mod.rs
pub mod config;
pub mod error;
pub mod models;
pub mod search;
pub mod sites;
pub mod utils;
pub mod export;

pub use config::*;
pub use error::*;
pub use models::*;
```

### Data Models
- Use `serde` for serialization
- Implement `Clone` and `Debug` traits
- Use enums to represent states

```rust
// ✅ Correct data models
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct SearchResult {
    pub site: String,
    pub url: String,
    pub status: SearchResultStatus,
    pub response_time: Option<u64>,
    pub metadata: Option<serde_json::Value>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum SearchResultStatus {
    Found,
    NotFound,
    Error,
    Timeout,
}
```

## 🚀 Performance Optimization

### Concurrency Control
- Use `Semaphore` to limit concurrency
- Use `tokio::spawn` for async tasks
- Set appropriate timeout values

```rust
// ✅ Correct concurrency control
pub async fn search_with_concurrency(
    sites: &[Site],
    max_concurrent: usize,
) -> Result<Vec<SearchResult>, SearchError> {
    let semaphore = Arc::new(Semaphore::new(max_concurrent));
    let mut handles = Vec::new();
    
    for site in sites {
        let semaphore = semaphore.clone();
        let handle = tokio::spawn(async move {
            let _permit = semaphore.acquire().await?;
            // Execute search logic
            Ok::<_, SearchError>(())
        });
        handles.push(handle);
    }
    
    // Wait for all tasks to complete
    let mut results = Vec::new();
    for handle in handles {
        if let Ok(result) = handle.await? {
            results.push(result);
        }
    }
    
    Ok(results)
}
```

### Memory Management
- Use `Arc` for reference counting
- Avoid unnecessary cloning
- Use `Cow` to handle strings

```rust
// ✅ Correct memory management
pub struct SearchEngine {
    config: Arc<SearchConfig>,
    client: Arc<reqwest::Client>,
}

impl SearchEngine {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [funnyzak/name-seeker](https://github.com/funnyzak/name-seeker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
