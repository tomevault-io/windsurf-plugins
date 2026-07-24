---
trigger: always_on
description: This file contains important instructions for working on the phpdoc-parser project.
---

# CLAUDE.MD - Instructions for AI Assistant

This file contains important instructions for working on the phpdoc-parser project.

## Project Overview

**phpstan/phpdoc-parser** is a library that represents PHPDocs with an Abstract Syntax Tree (AST). It supports parsing and modifying PHPDocs, and is primarily used by PHPStan for static analysis.

### Key Features
- Parses PHPDoc comments into an AST representation
- Supports all PHPDoc tags and types (see [PHPStan documentation](https://phpstan.org/writing-php-code/phpdocs-basics))
- Format-preserving printer for modifying and printing AST nodes
- Support for Doctrine Annotations parsing
- Nullable, intersection, generic, and conditional types support

### Requirements
- PHP ^7.4 || ^8.0
- Platform target: PHP 7.4.6

## Project Structure

### Source Code (`src/`)

The source code is organized into the following main components:

1. **Lexer** (`src/Lexer/`)
   - `Lexer.php` - Tokenizes PHPDoc strings

2. **Parser** (`src/Parser/`)
   - `PhpDocParser.php` - Main PHPDoc parser (parses tags and structure)
   - `TypeParser.php` - Parses PHPDoc type expressions
   - `ConstExprParser.php` - Parses constant expressions
   - `TokenIterator.php` - Iterator for tokens
   - `StringUnescaper.php` - Handles string unescaping
   - `ParserException.php` - Exception handling

3. **AST** (`src/Ast/`)
   - `Node.php` - Base AST node interface
   - `NodeTraverser.php` - Traverses and transforms AST
   - `NodeVisitor.php` - Visitor pattern for AST traversal
   - `Type/` - Type nodes (GenericTypeNode, ArrayTypeNode, UnionTypeNode, etc.)
   - `PhpDoc/` - PHPDoc tag nodes (ParamTagValueNode, ReturnTagValueNode, etc.)
   - `ConstExpr/` - Constant expression nodes
   - `NodeVisitor/` - Built-in visitors (CloningVisitor, etc.)

4. **Printer** (`src/Printer/`)
   - `Printer.php` - Prints AST back to PHPDoc format
   - `Differ.php` - Computes differences between AST nodes
   - `DiffElem.php` - Represents diff elements

5. **Configuration**
   - `ParserConfig.php` - Parser configuration (attributes to use)

### Tests (`tests/PHPStan/`)

Tests mirror the source structure and include:

1. **Parser Tests** (`tests/PHPStan/Parser/`)
   - `TypeParserTest.php` - Type parsing tests
   - `PhpDocParserTest.php` - PHPDoc parsing tests
   - `ConstExprParserTest.php` - Constant expression parsing tests
   - `FuzzyTest.php` - Fuzzy testing
   - `Doctrine/` - Doctrine annotation test fixtures

2. **AST Tests** (`tests/PHPStan/Ast/`)
   - `NodeTraverserTest.php` - Node traversal tests
   - `Attributes/AttributesTest.php` - AST attribute tests
   - `ToString/` - Tests for converting AST to string
   - `NodeVisitor/` - Visitor pattern tests

3. **Printer Tests** (`tests/PHPStan/Printer/`)
   - Tests for format-preserving printing functionality

### Configuration Files

- `phpunit.xml` - PHPUnit test configuration
- `phpstan.neon` - PHPStan static analysis configuration
- `phpstan-baseline.neon` - PHPStan baseline (known issues)
- `phpcs.xml` - PHP CodeSniffer configuration
- `composer.json` - Dependencies and autoloading

## How the Parser Works

The parsing flow follows these steps:

1. **Lexing**: `Lexer` tokenizes the PHPDoc string into tokens
2. **Parsing**: `PhpDocParser` uses `TypeParser` and `ConstExprParser` to build an AST
3. **Traversal/Modification**: `NodeTraverser` with `NodeVisitor` can traverse and modify the AST
4. **Printing**: `Printer` converts the AST back to PHPDoc format (optionally preserving formatting)

### Basic Usage Example

```php
$config = new ParserConfig(usedAttributes: []);
$lexer = new Lexer($config);
$constExprParser = new ConstExprParser($config);
$typeParser = new TypeParser($config, $constExprParser);
$phpDocParser = new PhpDocParser($config, $typeParser, $constExprParser);

$tokens = new TokenIterator($lexer->tokenize('/** @param Lorem $a */'));
$phpDocNode = $phpDocParser->parse($tokens);
```

### Format-Preserving Printing

For format-preserving printing (used when modifying existing PHPDocs), enable these attributes:
- `lines` - Preserve line information
- `indexes` - Preserve token indexes
- `comments` - Preserve comments

## Common Development Tasks

### Adding a New PHPDoc Tag
1. Create a new `*TagValueNode` class in `src/Ast/PhpDoc/`
2. Add parsing logic in `PhpDocParser.php`
3. Add tests in `tests/PHPStan/Parser/PhpDocParserTest.php`
4. Run tests and PHPStan

### Adding a New Type Node
1. Create a new `*TypeNode` class in `src/Ast/Type/`
2. Add parsing logic in `TypeParser.php`
3. Add printing logic in `Printer.php`
4. Add tests in `tests/PHPStan/Parser/TypeParserTest.php`
5. Run tests and PHPStan

### Modifying the Lexer
1. Update token generation in `Lexer.php`
2. Update parsers that consume those tokens
3. Add/update tests
4. Run comprehensive checks with `make check`

## Testing and Quality Checks

### Running Tests

Tests are run using PHPUnit:
```bash
make tests
```

Or directly:
```bash
php vendor/bin/phpunit
```

### Running PHPStan

PHPStan static analysis is run with:
```bash
make phpstan
```

Or directly:
```bash
php vendor/bin/phpstan
```

### Running All Checks

To run all quality checks (lint, code style, tests, and PHPStan):
```bash
make check
```

This runs:
- `lint` - PHP syntax checking with parallel-lint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phpstan/phpdoc-parser](https://github.com/phpstan/phpdoc-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
