---
trigger: always_on
description: Cursor rules for Xian Smart Contracts.
---

# Xian Smart Contract Development - Cursor Rules

XIAN is the currency of the Xian blockchain.
Never mention TAU or Lamden.

## Contract Structure

### Basic Structure
- Smart contracts are written in native Python without transpilation
- Contract names must follow the pattern: `^con_[a-z][a-z0-9_]*$`
- Contract names must start with 'con_' prefix (except system contracts like 'currency')
- Contract names must be lowercase, only contain letters, numbers and underscores after prefix
- Contract names must be max 64 characters

### Naming Conventions
- You cannot use '_' as a prefix for variables or functions (e.g., `_private_var` is not allowed)
- Follow standard Python naming conventions otherwise
- Use descriptive names for clarity
- A contract can not be deployed by another contract

### Function Types
- `@export` decorator defines public functions callable by any user or contract
- `@construct` decorator defines initialization function executed once at contract submission (optional)
- Functions without decorators are private and can only be called by the contract itself
- Functions with `@export` can call private functions internally

### Constructor Arguments
- Optional arguments can be provided to the `@construct` function
- Initial state can be setup using these arguments

## State Management

### Variable
- `Variable` is a way to define a singular state variable in the contract
- Use `variable.set(value)` to modify
- Use `variable.get()` to retrieve

```python
my_var = Variable()

@construct
def seed():
    my_var.set(0)  # Initialize variable

@export
def increment():
    my_var.set(my_var.get() + 1)
```

### Hash
- `Hash` is a key-value store for the contract
- Default value can be specified with `Hash(default_value=0)`
- Access through dictionary-like syntax: `hash[key] = value` and `hash[key]`
- Supports nested keys with tuple: `hash[key1, key2] = value`

```python
my_hash = Hash(default_value=0)

@export
def set_value(key: str, value: int):
    my_hash[key] = value

@export
def get_value(key: str):
    return my_hash[key]
```

#### Illegal Delimiters 
":" and "." cannot be used in Variable or Hash keys.

### Foreign State Access
- `ForeignHash` provides read-only access to a Hash from another contract
- `ForeignVariable` provides read-only access to a Variable from another contract

```python
token_balances = ForeignHash(foreign_contract='con_my_token', foreign_name='balances')
foundation_owner = ForeignVariable(foreign_contract='foundation', foreign_name='owner')
```

## Context Variables

### ctx.caller
- The identity of the person or contract calling the function
- Changes when a contract calls another contract's function
- Used for permission checks in token contracts

### ctx.signer
- The top-level user who signed the transaction
- Remains constant throughout transaction execution
- Only used for security guards/blacklisting, not for account authorization

### ctx.this
- The identity/name of the current contract
- Never changes
- Useful when the contract needs to refer to itself

### ctx.owner
- Owner of the contract, optional field set at time of submission
- Only the owner can call exported functions if set
- Can be changed with `ctx.owner = new_owner`

### ctx.entry
- Returns tuple of (contract_name, function_name) of the original entry point
- Helps identify what contract and function initiated the call chain

## Built-in Variables

### Time and Blockchain Information
- `now` - Returns the current datetime
- `block_num` - Returns the current block number, useful for block-dependent logic
- `block_hash` - Returns the current block hash, can be used as a source of randomness

Example usage:
```python
@construct
def seed():
    submission_time = Variable()
    submission_block_num = Variable()
    submission_block_hash = Variable()
    
    # Store blockchain state at contract creation
    submission_time.set(now)
    submission_block_num.set(block_num)
    submission_block_hash.set(block_hash)
```

## Imports and Contract Interaction

### Importing Contracts
- Use `importlib.import_module(contract_name)` for dynamic contract imports
- Static contract imports can be done with `import <contract_name>`
- Only use 'import' syntax for contracts, not for libraries or Python modules
- Trying to import standard libraries will not work within a contract (they're automatically available)
- Dynamic imports are preferred when the contract name is determined at runtime
- Can enforce interface with `importlib.enforce_interface()`
- NEVER import anything other than a contract.
- ALL contracting libraries are available globally
- NEVER IMPORT importlib. It is already available globally.

```python
@export
def interact_with_token(token_contract: str, recipient: str, amount: float):
    token = importlib.import_module(token_contract)
    
    # Define expected interface
    interface = [
        importlib.Func('transfer', args=('amount', 'to')),
        importlib.Var('balances', Hash)
    ]
    
    # Enforce interface
    assert importlib.enforce_interface(token, interface)
    
    # Call function on other contract
    token.transfer(amount=amount, to=recipient)
```

## Error Handling

### Assertions
- Use `assert` statements for validation and error checking

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XD3an/awesome-ai-coding-all-in-one](https://github.com/XD3an/awesome-ai-coding-all-in-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
