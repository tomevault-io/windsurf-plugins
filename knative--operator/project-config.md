---
trigger: always_on
description: - You can use ./yq with the `--debug-node-info` flag to get a deeper understanding of the ast.
---

# General rules
✅ **DO:**
- You can use ./yq with the `--debug-node-info` flag to get a deeper understanding of the ast.
- run ./scripts/format.sh to format the code; then ./scripts/check.sh lint and finally ./scripts/spelling.sh to check spelling.
- Add comprehensive tests to cover the changes
- Run test suite to ensure there is no regression
- Use UK english spelling

❌ **DON'T:**
- Git add or commit
- Add comments to functions that are self-explanatory



# Adding a New Encoder/Decoder

This guide explains how to add support for a new format (encoder/decoder) to yq without modifying `candidate_node.go`.

## Overview

The encoder/decoder architecture in yq is based on two main interfaces:

- **Encoder**: Converts a `CandidateNode` to output in a specific format
- **Decoder**: Reads input in a specific format and creates a `CandidateNode`

Each format is registered in `pkg/yqlib/format.go` and made available through factory functions.

## Architecture

### Key Files

- `pkg/yqlib/encoder.go` - Defines the `Encoder` interface
- `pkg/yqlib/decoder.go` - Defines the `Decoder` interface
- `pkg/yqlib/format.go` - Format registry and factory functions
- `pkg/yqlib/operator_encoder_decoder.go` - Encode/decode operators
- `pkg/yqlib/encoder_*.go` - Encoder implementations
- `pkg/yqlib/decoder_*.go` - Decoder implementations

### Interfaces

**Encoder Interface:**
```go
type Encoder interface {
    Encode(writer io.Writer, node *CandidateNode) error
    PrintDocumentSeparator(writer io.Writer) error
    PrintLeadingContent(writer io.Writer, content string) error
    CanHandleAliases() bool
}
```

**Decoder Interface:**
```go
type Decoder interface {
    Init(reader io.Reader) error
    Decode() (*CandidateNode, error)
}
```

## Step-by-Step: Adding a New Encoder/Decoder

### Step 1: Create the Encoder File

Create `pkg/yqlib/encoder_<format>.go` implementing the `Encoder` interface:
- `Encode()` - Convert a `CandidateNode` to your format and write to the output writer
- `PrintDocumentSeparator()` - Handle document separators if your format requires them
- `PrintLeadingContent()` - Handle leading content/comments if supported
- `CanHandleAliases()` - Return whether your format supports YAML aliases

See `encoder_json.go` or `encoder_base64.go` for examples.

### Step 2: Create the Decoder File

Create `pkg/yqlib/decoder_<format>.go` implementing the `Decoder` interface:
- `Init()` - Initialize the decoder with the input reader and set up any needed state
- `Decode()` - Decode one document from the input and return a `CandidateNode`, or `io.EOF` when finished

See `decoder_json.go` or `decoder_base64.go` for examples.

### Step 3: Create Tests (Mandatory)

Create a test file `pkg/yqlib/<format>_test.go` using the `formatScenario` pattern:
- Define test scenarios as `formatScenario` structs with fields: `description`, `input`, `expected`, `scenarioType`
- `scenarioType` can be `"decode"` (test decoding to YAML) or `"roundtrip"` (encode/decode preservation)
- Create a helper function `test<Format>Scenario()` that switches on `scenarioType`
- Create main test function `Test<Format>FormatScenarios()` that iterates over scenarios
- The main test function should use `documentScenarios` to ensure testcase documentation is generated.

Test coverage must include:
- Basic data types (scalars, arrays, objects/maps)
- Nested structures
- Edge cases (empty inputs, special characters, escape sequences)
- Format-specific features or syntax
- Round-trip tests: decode → encode → decode should preserve data

See `hcl_test.go` for a complete example.

### Step 4: Register the Format in format.go

Edit `pkg/yqlib/format.go`:

1. Add a new format variable:
   - `"<format>"` is the formal name (e.g., "json", "yaml")
   - `[]string{...}` contains short aliases (can be empty)
   - The first function creates an encoder (can be nil for encode-only formats)
   - The second function creates a decoder (can be nil for decode-only formats)

2. Add the format to the `Formats` slice in the same file

See existing formats in `format.go` for the exact structure.

### Step 5: Handle Encoder Configuration (if needed)

If your format has preferences/configuration options:

1. Create a preferences struct with your configuration fields
2. Update the encoder to accept preferences in its factory function
3. Update `format.go` to pass the configured preferences
4. Update `operator_encoder_decoder.go` if special indent handling is needed (see existing formats like JSON and YAML for the pattern)

This pattern is optional and only needed if your format has user-configurable options.

## Build Tags

Use build tags to allow optional compilation of formats:
- Add `//go:build !yq_no<format>` at the top of your encoder and decoder files
- Create a no-build version in `pkg/yqlib/no_<format>.go` that returns nil for encoder/decoder factories

This allows users to compile yq without certain formats using: `go build -tags yq_no<format>`

## Working with CandidateNode

The `CandidateNode` struct represents a YAML node with:
- `Kind`: The node type (ScalarNode, SequenceNode, MappingNode)
- `Tag`: The YAML tag (e.g., "!!str", "!!int", "!!map")
- `Value`: The scalar value (for ScalarNode only)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [knative/operator](https://github.com/knative/operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
