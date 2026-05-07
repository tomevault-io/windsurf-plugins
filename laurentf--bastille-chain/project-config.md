---
trigger: always_on
description: You are working on **Bastille**, a post-quantum blockchain written in Elixir with the following key characteristics:
---

# Bastille Blockchain - Cursor AI Rules

## Project Context
You are working on **Bastille**, a post-quantum blockchain written in Elixir with the following key characteristics:
- **Post-quantum cryptography**: Dilithium2, Falcon512, SPHINCS+
- **French Revolution theme**: 1789 block rewards, Bastille Day (July 14th) with 14 decimals
- **Utility token model**: No max supply (like DOGE), fixed 1789 BAST block rewards
- **Modern architecture**: 4-database storage (blocks, chain, state, index), GenServer-based
- **Mining**: Blake3 Proof-of-Work (low memory alternative to RandomX)

## Elixir Best Practices

### Code Style & Idioms
- **ALWAYS use the pipe operator** `|>` for data transformations (3+ function calls)
- **Pattern match in function heads** instead of `case`/`cond` when possible
- **Pattern match function parameters** for type safety and self-documenting code:
  ```elixir
  # ✅ Good - Clear contract, early type validation
  def add_block(%Block{} = block, %Chain{} = chain)
  def handle_call({:get_balance, address}, _from, %__MODULE__{} = state)
  def process_result({:ok, data}), do: transform(data)
  def process_result({:error, reason}), do: handle_error(reason)
  
  # ❌ Bad - Unclear types, manual validation needed
  def add_block(block, chain) when is_map(block)
  def handle_call(message, _from, state)
  def process_result(result) do
    case result do
      {:ok, data} -> transform(data)
      {:error, reason} -> handle_error(reason)
    end
  end
  ```
- **Use guards** for simple validations: `when is_binary(data) and byte_size(data) > 0`
- **Prefer `with`** over nested `case` statements for error handling
- **Use `Stream`** instead of `Enum` for large datasets or file operations
- **Leverage `tap/2`** for side effects in pipelines
- **Use `then/2`** for final transformations or return value wrapping

### Function Design
- **Single responsibility**: One function, one clear purpose
- **Pattern match function parameters**: Always use pattern matching when you know the expected shape
  - **Structs**: `def process_block(%Block{} = block)` not `def process_block(block)`
  - **Maps**: `def handle_config(%{host: host, port: port})` not `def handle_config(config)`
  - **Tuples**: `def handle_result({:ok, value})` and `def handle_result({:error, reason})`
  - **Lists**: `def process_items([head | tail])` when structure matters
  - **GenServer state**: `def handle_call(msg, _from, %__MODULE__{} = state)`
- **Pattern match on struct fields**: `%Transaction{signature_type: :coinbase}` in function heads
- **Use meaningful guard names**: `when is_valid_address(addr)` not `when byte_size(addr) == 44`
- **Return consistent tuples**: `{:ok, result}` or `{:error, reason}`
- **Avoid deep nesting**: Extract helper functions

### GenServer Patterns
- **Handle all message types** explicitly, avoid catch-all clauses
- **Use structured state**: Define `defstruct` for GenServer state
- **Pattern match messages** in function heads: `handle_call({:get, key}, _from, state)`
- **Separate business logic** from GenServer boilerplate
- **Use timeouts appropriately** for blockchain operations

### Error Handling
- **Use `with` for validation chains**: Multiple validations that can fail
- **Pattern match on errors**: `{:error, :not_found}` not generic `{:error, _}`
- **Provide meaningful error context**: `{:error, {:insufficient_balance, required: 100, available: 50}}`
- **Log errors with context**: Include relevant data for debugging

## Blockchain-Specific Guidelines

### Cryptographic Operations
- **Always validate inputs** before cryptographic operations
- **Use pattern matching** for key types: `%{dilithium: dil_key, falcon: fal_key, sphincs: sph_key}`
- **Handle key size validation** with constants from `Bastille.Core.Crypto`
- **Separate key generation** from address derivation logic

### Transaction Handling
- **Validate transactions atomically**: All-or-nothing validation
- **Use pattern matching** for transaction types: coinbase, regular, genesis
- **Handle nonce sequences** carefully to prevent replay attacks
- **Separate validation** from state application

### Storage Patterns
- **Use batch operations** for atomic writes across multiple databases
- **Pattern match storage results**: `{:ok, data}` vs `{:error, reason}`
- **Handle partitioned data** (time-based block storage) explicitly
- **Index frequently queried data** (block hashes, transaction hashes)

### Mining & Consensus
- **Handle mining results asynchronously**: Use GenServer messages
- **Validate blocks completely** before adding to chain
- **Separate mining logic** from validation logic
- **Handle consensus state transitions** explicitly

## Architecture Patterns

### Module Organization
```
lib/bastille/
├── core/           # Pure functions, data structures
├── economics/      # Tokenomics, constants, burn tracking
├── storage/        # Database interfaces (4-database architecture)
├── consensus/      # Mining, validation, difficulty adjustment
├── blockchain/     # Chain state management
├── mempool/        # Transaction pool
├── p2p/           # Network communication
├── rpc/           # JSON-RPC API endpoints
└── validator/     # Block production, mining coordination
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laurentf/bastille-chain](https://github.com/laurentf/bastille-chain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
