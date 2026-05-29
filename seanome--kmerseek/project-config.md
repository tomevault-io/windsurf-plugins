---
trigger: always_on
description: Write idiomatic Rust code with detailed explanations of WHY each choice follows Rust best practices
---


# Idiomatic Rust Development Guide

## Core Principle

Always write idiomatic Rust code as if you are a very experienced, seasoned Rustacean. For every code change or suggestion, explain WHY it's idiomatic Rust and what makes it follow Rust best practices. Focus on the reasoning behind the code, not just what it does.

## Error Handling Philosophy

**WHY**: Rust's error handling is designed to make failures explicit and composable. Unlike languages with exceptions, Rust forces you to handle errors at compile time, preventing silent failures that can crash scientific computations.

```rust
// Idiomatic: Explicit error handling with Result
fn process_sequence(data: &str) -> Result<Vec<u8>, ParseError> {
    data.chars()
        .map(|c| match c {
            'A' | 'a' => Ok(0),
            'T' | 't' => Ok(1),
            'G' | 'g' => Ok(2),
            'C' | 'c' => Ok(3),
            _ => Err(ParseError::InvalidBase(c)),
        })
        .collect() // collect() propagates the first error automatically
}
```

**Why this is idiomatic**:

- `Result<T, E>` makes error cases explicit in the type system
- The `?` operator (implicit in `collect()`) provides clean error propagation
- Pattern matching is exhaustive, preventing missed error cases
- No panics in library code - errors are recoverable

## Ownership and Borrowing Strategy

**WHY**: Rust's ownership system prevents data races and memory issues at compile time. Borrowing reduces allocations and makes APIs more flexible, crucial for performance-critical bioinformatics code.

```rust
// Idiomatic: Borrow when you don't need ownership
fn analyze_kmers(sequence: &str, k: usize) -> Vec<&str> {
    sequence
        .char_indices()
        .filter_map(|(i, _)| {
            if i + k <= sequence.len() {
                Some(&sequence[i..i + k])
            } else {
                None
            }
        })
        .collect()
}

// Idiomatic: Use Cow for flexible ownership
use std::borrow::Cow;
fn normalize_sequence(input: &str) -> Cow<'_, str> {
    if input.chars().all(|c| c.is_ascii_uppercase()) {
        Cow::Borrowed(input) // No allocation needed
    } else {
        Cow::Owned(input.to_uppercase()) // Only allocate when necessary
    }
}
```

**Why this is idiomatic**:

- `&str` parameter avoids unnecessary `String` allocations
- `Cow<'_, str>` provides zero-cost abstraction for owned vs borrowed data
- Slicing with `&sequence[i..i + k]` creates views without copying
- The borrow checker ensures memory safety without runtime overhead

## Parallel Processing with Rayon

**WHY**: Rayon provides data parallelism with minimal overhead. For bioinformatics workloads processing large sequences, parallelization is essential for performance.

```rust
use rayon::prelude::*;

// Idiomatic: Use rayon for parallel iteration
fn process_sequences_parallel(sequences: &[String]) -> Vec<Vec<u8>> {
    sequences
        .par_iter() // Parallel iteration
        .map(|seq| encode_sequence(seq))
        .collect()
}

// Idiomatic: Parallel processing with error handling
fn validate_sequences_parallel(sequences: &[String]) -> Result<Vec<bool>, ValidationError> {
    sequences
        .par_iter()
        .map(|seq| validate_sequence(seq))
        .collect::<Result<Vec<_>, _>>() // Collect Results in parallel
}

// Idiomatic: Parallel reduction for aggregations
fn count_total_kmers(sequences: &[String], k: usize) -> usize {
    sequences
        .par_iter()
        .map(|seq| count_kmers_in_sequence(seq, k))
        .sum() // Parallel reduction
}
```

**Why this is idiomatic**:

- `par_iter()` provides automatic work-stealing parallelism
- Rayon's parallel iterators have the same API as sequential ones
- `collect()` handles parallel collection efficiently
- Parallel reduction with `sum()` is much faster than sequential accumulation
- Rayon automatically handles thread pool management

## Memory Management for Genomic Data

**WHY**: Genomic datasets can be massive. Rust's zero-cost abstractions let you write high-level code without performance penalties, while the type system prevents memory leaks.

```rust
// Idiomatic: Pre-allocate with known capacity
fn build_kmer_index(sequences: &[String], k: usize) -> HashMap<String, Vec<usize>> {
    let estimated_kmers = sequences.len() * 1000; // Rough estimate
    let mut index = HashMap::with_capacity(estimated_kmers);

    for (seq_idx, sequence) in sequences.iter().enumerate() {
        for kmer in sequence.chars().collect::<Vec<_>>().windows(k) {
            let kmer_str: String = kmer.iter().collect();
            index.entry(kmer_str).or_insert_with(Vec::new).push(seq_idx);
        }
    }
    index
}

// Idiomatic: Use appropriate smart pointers
use std::sync::Arc;
use parking_lot::RwLock;

struct SharedIndex {
    data: Arc<RwLock<HashMap<String, Vec<usize>>>>,
}

impl SharedIndex {
    fn new() -> Self {
        Self {
            data: Arc::new(RwLock::new(HashMap::new())),
        }
    }

    fn insert(&self, key: String, value: Vec<usize>) {
        self.data.write().insert(key, value);
    }
}
```

**Why this is idiomatic**:

- `HashMap::with_capacity()` prevents multiple reallocations
- `Arc<RwLock<T>>` provides thread-safe shared ownership

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seanome/kmerseek](https://github.com/seanome/kmerseek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
