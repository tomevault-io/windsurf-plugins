---
trigger: always_on
description: This document contains knowledge about the React Compiler gathered during development sessions. It serves as a reference for understanding the codebase architecture and key concepts.
---

# React Compiler Knowledge Base

This document contains knowledge about the React Compiler gathered during development sessions. It serves as a reference for understanding the codebase architecture and key concepts.

## Project Structure

When modifying the compiler, you MUST read the documentation about that pass in `compiler/packages/babel-plugin-react-compiler/docs/passes/` to learn more about the role of that pass within the compiler.

- `packages/babel-plugin-react-compiler/` - Main compiler package
  - `src/HIR/` - High-level Intermediate Representation types and utilities
  - `src/Inference/` - Effect inference passes (aliasing, mutation, etc.)
  - `src/Validation/` - Validation passes that check for errors
  - `src/Entrypoint/Pipeline.ts` - Main compilation pipeline with pass ordering
  - `src/__tests__/fixtures/compiler/` - Test fixtures
    - `error.todo-*.js` - Unsupported feature, correctly throws Todo error (graceful bailout)
    - `error.bug-*.js` - Known bug, throws wrong error type or incorrect behavior
    - `*.expect.md` - Expected output for each fixture

## Running Tests

```bash
# Run all tests
yarn snap

# Run tests matching a pattern
# Example: yarn snap -p 'error.*'
yarn snap -p <pattern>

# Run a single fixture in debug mode. Use the path relative to the __tests__/fixtures/compiler directory
# For each step of compilation, outputs the step name and state of the compiled program
# Example: yarn snap -p simple.js -d
yarn snap -p <file-basename> -d

# Update fixture outputs (also works with -p)
yarn snap -u
```

## Linting

```bash
# Run lint on the compiler source
yarn workspace babel-plugin-react-compiler lint
```

## Formatting

```bash
# Run prettier on all files (from the react root directory, not compiler/)
yarn prettier-all
```

## Compiling Arbitrary Files

Use `yarn snap compile` to compile any file (not just fixtures) with the React Compiler:

```bash
# Compile a file and see the output
yarn snap compile <path>

# Compile with debug logging to see the state after each compiler pass
# This is an alternative to `yarn snap -d -p <pattern>` when you don't have a fixture file yet
yarn snap compile --debug <path>
```

## Minimizing Test Cases

Use `yarn snap minimize` to automatically reduce a failing test case to its minimal reproduction:

```bash
# Minimize a file that causes a compiler error
yarn snap minimize <path>

# Minimize and update the file in-place with the minimized version
yarn snap minimize --update <path>
```

## Version Control

This repository uses Sapling (`sl`) for version control. Sapling is similar to Mercurial: there is not staging area, but new/deleted files must be explicitly added/removed.

```bash
# Check status
sl status

# Add new files, remove deleted files
sl addremove

# Commit all changes
sl commit -m "Your commit message"

# Commit with multi-line message using heredoc
sl commit -m "$(cat <<'EOF'
Summary line

Detailed description here
EOF
)"
```

## Key Concepts

### HIR (High-level Intermediate Representation)

The compiler converts source code to HIR for analysis. Key types in `src/HIR/HIR.ts`:

- **HIRFunction** - A function being compiled
  - `body.blocks` - Map of BasicBlocks
  - `context` - Captured variables from outer scope
  - `params` - Function parameters
  - `returns` - The function's return place
  - `aliasingEffects` - Effects that describe the function's behavior when called

- **Instruction** - A single operation
  - `lvalue` - The place being assigned to
  - `value` - The instruction kind (CallExpression, FunctionExpression, LoadLocal, etc.)
  - `effects` - Array of AliasingEffects for this instruction

- **Terminal** - Block terminators (return, branch, etc.)
  - `effects` - Array of AliasingEffects

- **Place** - A reference to a value
  - `identifier.id` - Unique IdentifierId

- **Phi nodes** - Join points for values from different control flow paths
  - Located at `block.phis`
  - `phi.place` - The result place
  - `phi.operands` - Map of predecessor block to source place

### AliasingEffects System

Effects describe data flow and operations. Defined in `src/Inference/AliasingEffects.ts`:

**Data Flow Effects:**
- `Impure` - Marks a place as containing an impure value (e.g., Date.now() result, ref.current)
- `Capture a -> b` - Value from `a` is captured into `b` (mutable capture)
- `Alias a -> b` - `b` aliases `a`
- `ImmutableCapture a -> b` - Immutable capture (like Capture but read-only)
- `Assign a -> b` - Direct assignment
- `MaybeAlias a -> b` - Possible aliasing
- `CreateFrom a -> b` - Created from source

**Mutation Effects:**
- `Mutate value` - Value is mutated
- `MutateTransitive value` - Value and transitive captures are mutated
- `MutateConditionally value` - May mutate
- `MutateTransitiveConditionally value` - May mutate transitively

**Other Effects:**
- `Render place` - Place is used in render context (JSX props, component return)
- `Freeze place` - Place is frozen (made immutable)
- `Create place` - New value created
- `CreateFunction` - Function expression created, includes `captures` array
- `Apply` - Function application with receiver, function, args, and result


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [react/react](https://github.com/react/react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
