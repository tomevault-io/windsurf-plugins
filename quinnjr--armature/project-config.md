---
trigger: always_on
description: Guidelines for performance optimization and benchmarking in the Armature framework.
---


# Performance & Benchmarking

Guidelines for performance optimization and benchmarking in the Armature framework.

## Benchmark Infrastructure

### Running Benchmarks

```bash
# Run all benchmarks
cargo bench

# Run specific benchmark
cargo bench --bench core_benchmarks

# Run with native CPU optimizations
cargo bench --profile release-native

# Run with flamegraph profiling
cargo bench --profile profiling
```

### Benchmark Location

All benchmarks are in `benches/`:

```
benches/
├── core_benchmarks.rs        # Core framework benchmarks
├── security_benchmarks.rs    # Crypto/hashing benchmarks
├── validation_benchmarks.rs  # Input validation
├── cache_benchmarks.rs       # Cache operations
├── auth_benchmarks.rs        # Authentication
├── json_benchmarks.rs        # JSON parsing/serialization
├── http_client_benchmarks.rs # HTTP client
└── framework_comparison.rs   # Compare with Actix/Axum/Warp
```

## Writing Benchmarks with Criterion

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion, BenchmarkId};

fn benchmark_router(c: &mut Criterion) {
    let router = Router::new()
        .route("/users", get(users_handler))
        .route("/users/:id", get(user_handler));

    c.bench_function("router_static_route", |b| {
        b.iter(|| {
            black_box(router.match_route("/users", Method::GET))
        })
    });

    c.bench_function("router_dynamic_route", |b| {
        b.iter(|| {
            black_box(router.match_route("/users/123", Method::GET))
        })
    });
}

// Parameterized benchmarks
fn benchmark_json_parsing(c: &mut Criterion) {
    let mut group = c.benchmark_group("json_parsing");

    for size in [100, 1000, 10000].iter() {
        let json = generate_json(*size);

        group.bench_with_input(
            BenchmarkId::from_parameter(size),
            &json,
            |b, json| {
                b.iter(|| {
                    black_box(serde_json::from_str::<Value>(json).unwrap())
                })
            },
        );
    }

    group.finish();
}

criterion_group!(benches, benchmark_router, benchmark_json_parsing);
criterion_main!(benches);
```

## Async Benchmarks

```rust
use criterion::{criterion_group, criterion_main, Criterion};
use tokio::runtime::Runtime;

fn benchmark_async_handler(c: &mut Criterion) {
    let rt = Runtime::new().unwrap();

    c.bench_function("async_handler", |b| {
        b.to_async(&rt).iter(|| async {
            let response = handle_request().await;
            black_box(response)
        })
    });
}
```

## Profiling

### CPU Profiling with Flamegraph

```bash
# Install flamegraph
cargo install flamegraph

# Generate flamegraph
cargo flamegraph --bench core_benchmarks -- --bench

# Or for a running server
cargo flamegraph --example profiling_server
```

### Memory Profiling

The project has comprehensive memory profiling tools:

```bash
# Use the memory profiling script
./scripts/memory-profile.sh dhat 30      # DHAT (recommended for Rust)
./scripts/memory-profile.sh valgrind 30  # Valgrind leak detection
./scripts/memory-profile.sh massif 30    # Massif heap profiler
./scripts/memory-profile.sh heaptrack 30 # Heaptrack detailed analysis

# Build with DHAT support
cargo build --example memory_profile_server --release --features memory-profiling

# Run memory benchmarks
cargo bench --bench memory_benchmarks
```

**DHAT Setup:**

```rust
#[cfg(feature = "memory-profiling")]
#[global_allocator]
static ALLOC: dhat::Alloc = dhat::Alloc;

fn main() {
    #[cfg(feature = "memory-profiling")]
    let _profiler = dhat::Profiler::new_heap();
    // Run workload - report generated on exit
}
```

View DHAT reports at: https://nnethercote.github.io/dh_view/dh_view.html

See `docs/memory-profiling-guide.md` for complete documentation.

### Using perf

```bash
# Record performance data
perf record -g cargo bench --bench core_benchmarks

# Generate report
perf report

# Generate flamegraph from perf data
perf script | stackcollapse-perf.pl | flamegraph.pl > flamegraph.svg
```

## Build Profiles

The project has optimized build profiles in `Cargo.toml`:

| Profile | Use Case | LTO | Optimizations |
|---------|----------|-----|---------------|
| `release` | Standard release | thin | O3 |
| `release-fat` | Maximum optimization | fat | O3 + panic=abort |
| `release-native` | Benchmarks | thin | O3 + target-cpu=native |
| `profiling` | Profiling | thin | O3 + debug symbols |
| `pgo-generate` | PGO data collection | thin | O3 |
| `pgo-use` | PGO-optimized build | fat | O3 |

### Profile-Guided Optimization (PGO)

```bash
# Step 1: Build with profiling instrumentation
RUSTFLAGS="-Cprofile-generate=/tmp/pgo" cargo build --profile pgo-generate

# Step 2: Run representative workload
./target/pgo-generate/armature-benchmark

# Step 3: Merge profile data
llvm-profdata merge -o merged.profdata /tmp/pgo/*.profraw

# Step 4: Build with PGO
RUSTFLAGS="-Cprofile-use=$(pwd)/merged.profdata" cargo build --profile pgo-use
```

## Performance Patterns

### Zero-Cost Abstractions

```rust
// ✅ Good: Zero-cost abstraction with generics
pub fn process<T: AsRef<[u8]>>(data: T) -> Result<(), Error> {
    let bytes = data.as_ref();
    // Process bytes
    Ok(())
}

// ❌ Bad: Dynamic dispatch when not needed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/armature](https://github.com/quinnjr/armature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
