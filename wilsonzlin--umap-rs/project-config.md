---
trigger: always_on
description: `src/layout/optimize_layout_euclidean.rs:30-57` implements `UnsafeSyncCell<T>` which **intentionally allows data races**. This is correct for parallel SGD:
---

# Notes for future agents/developers

## Critical things that look wrong but are correct

### UnsafeSyncCell and data races
`src/layout/optimize_layout_euclidean.rs:30-57` implements `UnsafeSyncCell<T>` which **intentionally allows data races**. This is correct for parallel SGD:

- Multiple threads write to the same embedding array without locks
- Races occur when two edges share a vertex (rare)
- Lost updates are acceptable because SGD is stochastic
- This matches Python UMAP's Numba `parallel=True` behavior
- Academic justification: Hogwild! algorithm (Recht et al., 2011)

**Do NOT "fix" this by adding locks/atomics.** Performance would tank and it's unnecessary.

### ParallelVec and UnsafeCell usage
`src/umap/fuzzy_simplicial_set.rs` uses `ParallelVec<T>` for parallel CSR construction. This uses `std::cell::UnsafeCell<Vec<T>>` internally, which is **required** for correct unsafe code:

- `UnsafeCell<T>` is Rust's primitive for interior mutability
- It tells the compiler "the T inside may be mutated through shared references"
- Without it, the compiler assumes `&Vec<T>` means immutable contents and may optimize incorrectly (UB)
- **Correct pattern**: `UnsafeCell<Vec<T>>` - the Vec's contents may be mutated
- **Wrong pattern**: `UnsafeCell<*mut T>` - only wraps the pointer, not the data

```rust
struct ParallelVec<T> {
  data: UnsafeCell<Vec<T>>,  // Vec contents may be mutated through &self
}
unsafe impl<T: Send> Sync for ParallelVec<T> {}  // Safe if disjoint writes

impl<T> ParallelVec<T> {
  unsafe fn write(&self, index: usize, value: T) {
    let vec = &mut *self.data.get();  // UnsafeCell::get() → *mut Vec<T>
    *vec.get_unchecked_mut(index) = value;
  }
  fn into_inner(self) -> Vec<T> {
    self.data.into_inner()  // Extract Vec after parallel work
  }
}
```

**Key invariant**: Each row i writes only to `[indptr[i]..indptr[i+1]]`, which are disjoint by construction of the prefix sum.

### Lifetime patterns in fuzzy_simplicial_set.rs
`FuzzySimplicialSet<'a, 'd>` has two lifetimes but no bound between them. This is intentional:

- `'a` is for KNN data (indices/distances)
- `'d` is for disconnections set (DashSet)
- They're independent - no relationship needed
- Tried `'d: 'a` - causes borrow checker errors in caller

**Do NOT add lifetime bounds without testing the full call chain.**

### ArrayView2 passed by value
Throughout the codebase, `ArrayView2<'a, T>` is passed by value, not reference:

```rust
fn foo(distances: ArrayView2<'a, f32>) { ... }  // Correct
// NOT: fn foo(distances: &ArrayView2<'a, f32>) { ... }
```

`ArrayView2` is a thin wrapper (pointer + shape metadata), cheap to copy. Passing by value is idiomatic for ndarray views.

### No TypedBuilder struct in optimize_layout_generic.rs
Earlier version tried using a `TypedBuilder` struct for single-epoch optimization. This failed due to lifetime issues (can't borrow embeddings mutably multiple times through struct fields).

Solution: standalone function with many parameters. Yes, it triggers `clippy::too_many_arguments`. That's fine - the alternative doesn't compile.

**Do NOT try to refactor into a builder struct without solving the lifetime puzzle.**

## Performance-critical sections

### Parallel loop in optimize_layout_euclidean.rs:338-436
This is the hot path. ~80% of runtime is here. Changes here affect performance directly:

- Keep allocations outside the parallel loop if possible
- `Vec::with_capacity(dim)` is acceptable inside (stack-like, tiny)
- Do NOT add synchronization primitives
- Do NOT call external functions that aren't inline

### Direct CSR construction in fuzzy_simplicial_set.rs
Graph construction uses direct CSR building instead of TriMat/COO to avoid OOM on large datasets:

1. **Count phase**: Parallel count of entries per row
2. **Indptr phase**: Sequential prefix sum (fast, O(n))
3. **Fill phase**: Parallel fill, each row writes to `[indptr[i]..indptr[i+1]]`
4. **Sort phase**: Parallel per-row sort (insertion sort, O(k²) for k~256)

This avoids:
- Allocating O(nnz) intermediate triplet arrays
- O(nnz log nnz) global sort (replaced by O(n × k log k) local sorts)
- Multiple copies during format conversion

**Memory**: Only stores the final CSR arrays (indptr, indices, data) plus temporary row counts. No intermediate triplet/COO storage.

### u32 indices and CSC structure-only optimization
For datasets up to ~4 billion samples, `SparseMat` uses `u32` indices instead of `usize`:
- Halves index memory (4 bytes vs 8 bytes per entry)
- `CsMatI<f32, u32>` stores indptr and indices as `Vec<u32>`

The CSC (transpose) representation stores only structure (indptr + indices), not data:
- Avoids duplicating the data array entirely
- Values are looked up in original CSR via binary search O(log k), where k ≈ 256
- This is fast enough since k is small

### Distance calculations
Both Euclidean implementations use squared distance to avoid sqrt:

```rust
let dist_squared = rdist(&current, &other);  // No sqrt
// Then use dist_squared directly in formulas
```

**Do NOT add sqrt calls.** The formulas are designed for squared distance.

### Parallel patterns that avoid allocator contention


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wilsonzlin/umap-rs](https://github.com/wilsonzlin/umap-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
