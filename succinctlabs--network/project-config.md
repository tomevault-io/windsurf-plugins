---
trigger: always_on
description: - Group imports by source: `std`, external crates, then local modules
---

# Claude Coding Style Guidelines for Rust

## Import Organization
- Group imports by source: `std`, external crates, then local modules
- Use explicit imports rather than glob imports (avoid `use foo::*`)
- Organize external crate imports alphabetically
- Place local module imports (`use crate::`) at the bottom of imports
- Separate import groups with blank lines

Example:
```rust
use std::collections::HashMap;

use alloy_primitives::{Address, U256};
use anyhow::Result;
use serde::{Deserialize, Serialize};
use tracing::info;

use crate::{
    actions::Action,
    errors::{VAppError, VAppPanic},
    events::VAppEvent,
};
```

## Documentation
- Use triple-slash comments (`///`) for public API documentation
- Document all public structs, enums, functions, and methods
- Include examples in documentation when helpful
- Use square brackets for parameter references in docs: `[VAppEvent]`, `[State]`
- Document error conditions and panics

## Comments and Code Style
In this codebase, comments are always written in full sentences with periods.

Try to follow the style similar to:

```rust
/// Main event processing loop with retry logic.
pub async fn run(&mut self) -> Result<(), VAppDriverError> {
    let mut interval = time::interval(Duration::from_millis(self.config.event_interval_ms));
    loop {
        // Wait for the interval to tick.
        interval.tick().await;

        // Fetch txs.
        let txs = self.fetch(self.cursor).await?;

        // If there are no new events, sleep until the next interval.
        if txs.is_empty() {
            debug!("no new events found, sleeping until next interval");
            continue;
        }

        // Process the transactions.
        for (pos, tx) in txs.iter() {
            // Check that the cursor is correct.
            if pos != &self.cursor {
                return Err(VAppDriverError::CursorMismatch {
                    expected: self.cursor,
                    got: *pos,
                });
            }

            // Execute the transaction.
            match self.current.execute::<MockVerifier>(tx) {
                Ok(Some(action)) => {
                    info!("tx {} processed with action: {:?}", pos, action);
                }
                Ok(None) => {
                    info!("tx {} processed", pos);
                }
                Err(VAppError::Revert(revert)) => {
                    warn!("tx {} reverted: {:?}", pos, revert);
                }
                Err(VAppError::Panic(panic)) => {
                    return Err(VAppDriverError::VAppPanic(panic));
                }
            }

            // Add the transaction to the mempool.
            self.mempool.push((*pos, tx.clone()));

            // Every `prover_interval` transactions, generate a new block proof.
            if self.cursor % self.config.prover_interval == 0 {
                let proof = if self.config.debug {
                    // Load proof from disk in debug mode.
                    let proof_bytes = std::fs::read("proof.bin").unwrap();
                    bincode::deserialize(&proof_bytes).unwrap()
                } else {
                    // Generate the proof.
                    let txs = std::mem::take(&mut self.mempool);
                    let proof = self.prove(&self.prev, txs).await?;

                    // Save to disk in case we want to debug faster.
                    let proof_bytes = bincode::serialize(&proof).unwrap();
                    std::fs::write("proof.bin", proof_bytes).unwrap();

                    proof
                };

                // Submit the proof to the vApp.
                self.submit(proof).await?;

                // Update the previous state.
                self.prev = self.current.clone();
            }

            // Update the cursor to the next position.
            self.cursor += 1;
        }
    }
}
```

## Error Handling
- Use `thiserror::Error` for custom error types
- Implement comprehensive error variants with descriptive messages
- Use `#[from]` attribute for automatic error conversion
- Include context in error messages (expected vs actual values)
- Use `Result<T, ErrorType>` consistently for fallible operations

Example:
```rust
#[derive(Debug, Error)]
pub enum VAppDriverError {
    #[error("cursor mismatch: expected {expected}, got {got}")]
    CursorMismatch { expected: i64, got: i64 },
    
    #[error("database query error: {0}")]
    DatabaseQueryError(#[from] sqlx::Error),
}
```

## Struct and Field Organization
- Place public fields before private fields
- Use descriptive field names with full words (not abbreviations)
- Group related fields together
- Document complex fields with inline comments
- Use `#[derive(Debug, Clone, PartialEq, Eq, Serialize, Deserialize)]` for data structures

## Function Style
- Use descriptive function names with full words
- Place `pub` functions before private functions
- Use `&self` for read-only methods, `&mut self` for mutating methods
- Accept borrowed parameters (`&str`, `&[T]`) when ownership isn't needed
- Return `Result<T, E>` for fallible operations

## Logging
- Use `tracing` crate for logging (`info!`, `warn!`, `error!`, `debug!`)
- Include relevant context in log messages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [succinctlabs/network](https://github.com/succinctlabs/network) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
