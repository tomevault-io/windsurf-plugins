---
trigger: always_on
description: **Project**: Transmutation - High-performance document conversion engine for AI/LLM embeddings
---

# AI Development Rules for Transmutation

**Project**: Transmutation - High-performance document conversion engine for AI/LLM embeddings  
**Language**: Rust 1.85+ (edition 2024)  
**License**: MIT  
**Repository**: https://github.com/hivellm/transmutation

---

## Project Overview

Transmutation is a **pure Rust** document conversion engine designed to transform various file formats into optimized text and image outputs suitable for LLM processing and vector embeddings. This is a **high-performance alternative to Docling**, offering superior speed, lower memory usage, and zero runtime dependencies.

**Core Goals**:
- 100% Pure Rust implementation (no Python dependencies)
- Convert documents to LLM-friendly formats (Markdown, Images, JSON)
- Optimize output for embedding generation (text and multimodal)
- Maintain maximum quality with minimum size
- Faster and more efficient than Docling
- Seamless integration with HiveLLM Vectorizer

---

## Code Style

### Formatting
- Follow **Rust 2021/2024 edition** conventions
- Use `cargo fmt` with project-specific `rustfmt.toml`
- Maximum line length: **100 characters**
- Indentation: **4 spaces** (no tabs)
- Use trailing commas in multi-line lists/structs
- Group imports: std → external → crate → module

### Naming Conventions
- **Crates/Modules**: `snake_case` (e.g., `pdf_parser`, `image_ocr`)
- **Files**: `snake_case` (e.g., `pdf.rs`, `file_detect.rs`)
- **Structs/Enums/Traits**: `PascalCase` (e.g., `Converter`, `OutputFormat`, `DocumentConverter`)
- **Functions/Variables**: `snake_case` (e.g., `convert_to_markdown()`, `file_path`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `MAX_CHUNK_SIZE`, `DEFAULT_DPI`)
- **Type Parameters**: Single letter or `PascalCase` (e.g., `T`, `Item`, `Error`)

### Code Organization
- One module per file format converter (e.g., `pdf.rs`, `docx.rs`)
- Traits in `converters/traits.rs`
- Shared utilities in `utils/`
- Output format handlers in `output/`
- Error types in `error.rs`
- Public API in `lib.rs`

---

## Documentation

### Doc Comments
- **All public APIs MUST have doc comments** (`///` or `//!`)
- Use doc sections: `# Arguments`, `# Returns`, `# Errors`, `# Examples`, `# Panics`
- Provide runnable examples in doc tests
- Include links to related types/functions with `[Type]`

**Example**:
```rust
/// Converts a document to the specified output format.
///
/// This function handles the complete conversion workflow including
/// file detection, format validation, conversion, and optimization.
///
/// # Arguments
///
/// * `input_path` - Path to the input document
/// * `output_format` - Desired output format (Markdown, JSON, etc.)
/// * `options` - Conversion options for customization
///
/// # Returns
///
/// A `ConversionResult` containing the converted data and metadata
///
/// # Errors
///
/// * `FileNotFound` - If the input file does not exist
/// * `UnsupportedFormat` - If the file format is not supported
/// * `ConversionError` - If conversion fails
///
/// # Examples
///
/// ```rust
/// # use transmutation::{convert_document, OutputFormat, ConversionOptions};
/// # async fn example() -> Result<(), Box<dyn std::error::Error>> {
/// let result = convert_document(
///     "document.pdf",
///     OutputFormat::Markdown,
///     ConversionOptions::default()
/// ).await?;
/// # Ok(())
/// # }
/// ```
pub async fn convert_document(
    input_path: &str,
    output_format: OutputFormat,
    options: ConversionOptions,
) -> Result<ConversionResult> {
    // Implementation
}
```

### Module Documentation
- Add module-level docs (`//!`) at the top of each file
- Explain the purpose and main functionality
- Provide usage examples

### Project Documentation
- Update `docs/ROADMAP.md` after completing tasks
- Update `docs/CHANGELOG.md` for all user-facing changes
- Update `README.md` for major features
- Never create unnecessary `.md` files - consolidate in existing docs

---

## Testing Standards

### Test Organization
```
tests/
├── unit/              # Inline unit tests (#[cfg(test)] mod tests)
├── integration/       # Integration tests
└── fixtures/          # Test data (sample PDFs, DOCX, images, etc.)
```

### Coverage Requirements
- **Overall**: > 90%
- **Critical paths** (converters, parsers): 100%
- **Unit tests**: > 95%
- **Integration tests**: > 85%

### Test Naming
```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[tokio::test]
    async fn test_pdf_to_markdown_success() {
        // Arrange
        let input = "tests/fixtures/sample.pdf";
        let converter = PdfConverter::new();
        
        // Act
        let result = converter.to_markdown(input).await;
        
        // Assert
        assert!(result.is_ok());
        assert!(!result.unwrap().is_empty());
    }
    
    #[test]
    fn test_unsupported_file_format() {
        let result = detect_file_format("test.unknown");
        assert!(matches!(result, Err(Error::UnsupportedFormat(_))));
    }
}
```

### Run Tests Before Committing
```bash
# Run all tests
cargo test

# Run with output
cargo test -- --nocapture

# Run specific test

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hivellm/transmutation](https://github.com/hivellm/transmutation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
