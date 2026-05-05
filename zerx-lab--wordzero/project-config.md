---
trigger: always_on
description: WordZero is a Golang-based Word document (.docx) manipulation library that follows the Office Open XML (OOXML) specifications. The library provides comprehensive functionality for creating, reading, and modifying Word documents with a focus on performance and ease of use.
---

# Copilot Instructions for WordZero

## Project Overview

WordZero is a Golang-based Word document (.docx) manipulation library that follows the Office Open XML (OOXML) specifications. The library provides comprehensive functionality for creating, reading, and modifying Word documents with a focus on performance and ease of use.

## Core Philosophy

- **Zero Dependencies**: Pure Go implementation with minimal external dependencies (only goldmark for Markdown support)
- **Performance First**: Optimized for speed - average 2.62ms processing time
- **Clean API**: Fluent interface design for intuitive usage
- **Office Open XML Compliance**: Strict adherence to OOXML specifications

## Code Conventions

### General Guidelines

1. **Language**: 
   - Code comments should be in Chinese (中文) as per project standard
   - Function/method documentation should include Chinese descriptions
   - Variable names and code should use English

2. **Naming Conventions**:
   - Use PascalCase for exported types and functions
   - Use camelCase for unexported types and functions
   - Constants should use PascalCase or SCREAMING_SNAKE_CASE where appropriate

3. **Code Organization**:
   - Keep related functionality in the same file
   - Main types: `Document`, `Paragraph`, `Table`, `Run`, etc.
   - Package structure:
     - `pkg/document/` - Core document manipulation
     - `pkg/style/` - Style management
     - `pkg/markdown/` - Markdown conversion

4. **Error Handling**:
   - Use custom error types from `errors.go`
   - Wrap errors with context using `WrapError` or `WrapErrorWithContext`
   - Return meaningful error messages

### Go-Specific Guidelines

1. **Interfaces**: Define interfaces where abstraction is needed (e.g., `BodyElement`)

2. **Struct Tags**: Use XML struct tags for OOXML serialization:
   ```go
   type Paragraph struct {
       XMLName    xml.Name             `xml:"w:p"`
       Properties *ParagraphProperties `xml:"w:pPr,omitempty"`
       Runs       []Run                `xml:"w:r"`
   }
   ```

3. **Method Receivers**: Use pointer receivers for methods that modify the receiver

4. **XML Marshaling**: Implement custom `MarshalXML` methods when element ordering matters (see `Body.MarshalXML`)

## Project Structure

```
wordZero/
├── pkg/
│   ├── document/       # Core document manipulation API
│   ├── style/          # Style management system
│   └── markdown/       # Markdown ↔ Word conversion
├── examples/           # Usage examples and demos
├── benchmark/          # Performance benchmarks
├── test/              # Integration tests
└── docs/              # Documentation
```

## Building and Testing

### Build Commands

```bash
# Build all packages
go build ./...

# Build specific package
go build ./pkg/document
```

### Test Commands

```bash
# Run all tests
go test ./...

# Run tests for specific packages
go test ./pkg/document ./pkg/style ./test

# Run tests with coverage
go test -cover ./...

# Run short tests (skip long-running tests)
go test -short ./...

# Verbose test output
go test -v ./pkg/...
```

### Code Quality

```bash
# Format code
go fmt ./...

# Vet code for issues
go vet ./...

# Run linters (if golangci-lint is installed)
golangci-lint run
```

## Key Features to Understand

### 1. Document Operations
- Create, open, and save Word documents
- Add paragraphs, headings, and formatted text
- Support for document properties and metadata

### 2. Style System
- 18 predefined styles
- Custom style creation and inheritance
- Style manager for centralized style handling

### 3. Table Operations
- Complete table creation and manipulation
- Cell merging (horizontal, vertical, range)
- Cell formatting and styling
- Table iterators for traversal

### 4. Page Settings
- Page size (A4, Letter, Legal, etc.)
- Page orientation (portrait/landscape)
- Margins, headers, footers
- Section properties

### 5. Advanced Features
- Table of contents (TOC) generation
- Footnotes and endnotes
- List numbering (ordered/unordered)
- Template engine with inheritance
- Image embedding and manipulation
- Markdown ↔ Word bidirectional conversion

### 6. Template Engine
- Variable substitution: `{{variable}}`
- Conditionals: `{{#if condition}}...{{/if}}`
- Loops: `{{#each list}}...{{/each}}`
- Template inheritance: `{{extends "base"}}` with `{{#block}}` overrides

## Testing Standards

1. **Test File Naming**: Use `_test.go` suffix
2. **Test Function Naming**: Use `Test` prefix followed by function name
3. **Test Organization**: 
   - Unit tests in `pkg/*/` directories
   - Integration tests in `test/` directory
4. **Test Output**: Write test output files to `test_output/` directory
5. **Cleanup**: Always defer cleanup of test files using `os.RemoveAll`

Example:
```go
func TestCreateDocument(t *testing.T) {
    doc := document.New()
    doc.AddParagraph("Test content")
    
    err := doc.Save("test_output/test.docx")
    if err != nil {
        t.Fatalf("Failed to save: %v", err)
    }
    
    defer os.RemoveAll("test_output")
}
```

## API Design Patterns

### Fluent Interface

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zerx-lab/wordZero](https://github.com/zerx-lab/wordZero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
