---
trigger: always_on
description: PathCollab is a **collaborative digital pathology viewer** enabling real-time presenter-led sessions where pathologists can share whole slide images (WSIs) with synchronized navigation, annotations, and cursor tracking.
---

# PathCollab - Agent Instructions

## Project Vision

PathCollab is a **collaborative digital pathology viewer** enabling real-time presenter-led sessions where pathologists can share whole slide images (WSIs) with synchronized navigation, annotations, and cursor tracking.

### Core Principles

1. **Performance is a feature** - Sub-100ms tile serving, real-time cursor sync
2. **Stripe-level UX** - Every interaction should feel instant and delightful
3. **Correctness under load** - 1000s of concurrent connections without degradation

---

## Running the Project

### Frontend (React + Vite)
```bash
cd ./web && bun run dev --port 3000
```

### Backend (Rust + Axum)
```bash
SLIDES_DIR=/data/wsi_slides \
PORT=8080 \
RUST_LOG=pathcollab=debug,tower_http=debug \
cargo run --release  # Always use --release for perf work
```

---

## Performance Requirements

### Hard Budgets (Non-Negotiable)

| Operation | P50 | P99 | Notes |
|-----------|-----|-----|-------|
| Tile serving | < 30ms | < 100ms | Including decode + resize + encode |
| Cursor broadcast | < 20ms | < 100ms | End-to-end WebSocket delivery |
| Viewport sync | < 50ms | < 150ms | Presenter → all viewers |
| Initial slide load | < 500ms | < 1s | Metadata + first tiles |

### Scaling Targets

- **Concurrent connections**: 1000+ per session
- **Tile throughput**: 10,000 tiles/sec at P99 < 100ms
- **Memory per connection**: < 50KB baseline

---

## Backend Guidelines (Rust)

### Code Style

```rust
// ✅ DO: Use zero-copy patterns
fn process_tile(data: &[u8]) -> Result<Bytes, TileError> {
    // Work with references, return owned only at boundaries
}

// ❌ DON'T: Unnecessary allocations in hot paths
fn process_tile(data: Vec<u8>) -> Result<Vec<u8>, TileError> {
    // Copies on every call
}
```

### Async Patterns

```rust
// ✅ DO: Spawn blocking work correctly
let tile = tokio::task::spawn_blocking(move || {
    decode_tile(&path, region)  // CPU-bound work
}).await??;

// ✅ DO: Use buffered channels for backpressure
let (tx, rx) = tokio::sync::mpsc::channel(32);

// ❌ DON'T: Block the async runtime
let tile = decode_tile(&path, region);  // Blocks executor!

// ❌ DON'T: Unbounded channels in production
let (tx, rx) = tokio::sync::mpsc::unbounded_channel();
```

### Error Handling

```rust
// ✅ DO: Typed errors with context
#[derive(Debug, thiserror::Error)]
pub enum TileError {
    #[error("slide not found: {slide_id}")]
    SlideNotFound { slide_id: String },

    #[error("region out of bounds: {region:?} exceeds {dimensions:?}")]
    RegionOutOfBounds { region: Region, dimensions: Dimensions },

    #[error("decode failed: {source}")]
    DecodeFailed { #[from] source: image::ImageError },
}

// ✅ DO: Propagate with context
tile_cache.get(&key)
    .ok_or_else(|| TileError::SlideNotFound { slide_id: key.slide_id.clone() })?;
```

### Concurrency Patterns

```rust
// ✅ DO: Use DashMap for concurrent state
use dashmap::DashMap;
let sessions: DashMap<SessionId, Session> = DashMap::new();

// ✅ DO: Scope locks narrowly
{
    let session = sessions.get(&id).ok_or(Error::NotFound)?;
    session.connection_count()  // Lock released here
}

// ❌ DON'T: Hold locks across await points
let session = sessions.get(&id)?;
some_async_operation().await;  // Deadlock risk!
session.do_something();
```

### Performance Instrumentation

Always instrument new endpoints and hot paths:

```rust
use metrics::{histogram, counter};

async fn serve_tile(/* ... */) -> Result<Response, Error> {
    let start = std::time::Instant::now();

    let result = do_work().await;

    histogram!("pathcollab_tile_duration_seconds").record(start.elapsed());
    counter!("pathcollab_tiles_served_total").increment(1);

    result
}
```

### Caching Strategy

```rust
// Tile cache: LRU with size-based eviction
// Key insight: decoded tiles are larger than encoded, cache encoded

// ✅ DO: Cache at the right layer
cache_encoded_jpeg(&tile_key, &jpeg_bytes);  // ~50KB per tile

// ❌ DON'T: Cache decoded pixels
cache_decoded(&tile_key, &rgba_pixels);  // ~1MB per tile at 512x512
```

---

## Frontend Guidelines (React)

### Design Philosophy: Stripe-Level UX

1. **Instant feedback** - Every click has immediate visual response
2. **Optimistic updates** - Update UI before server confirms
3. **Graceful degradation** - Never show broken states
4. **Microinteractions** - Subtle animations that feel polished

### Component Patterns

```tsx
// ✅ DO: Optimistic UI with rollback
function useViewportSync() {
  const [viewport, setViewport] = useState(initialViewport);
  const [pending, setPending] = useState<Viewport | null>(null);

  const updateViewport = useCallback((newViewport: Viewport) => {
    // Immediate local update
    setViewport(newViewport);
    setPending(newViewport);

    // Send to server
    ws.send({ type: 'viewport', data: newViewport });
  }, [ws]);

  // Handle server rejection (rare)
  useEffect(() => {
    ws.on('viewport_rejected', (serverViewport) => {
      setViewport(serverViewport);  // Rollback
      toast.error('Viewport sync failed');
    });
  }, [ws]);


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PABannier/PathCollab](https://github.com/PABannier/PathCollab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
