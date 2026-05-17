---
trigger: always_on
description: **MCP for Beginners** is an open-source educational curriculum for learning the Model Context Protocol (MCP) - a standardized framework for interactions between AI models and client applications. This repository provides comprehensive learning materials with hands-on code examples across multiple programming languages.
---

# AGENTS.md

## Project Overview

**MCP for Beginners** is an open-source educational curriculum for learning the Model Context Protocol (MCP) - a standardized framework for interactions between AI models and client applications. This repository provides comprehensive learning materials with hands-on code examples across multiple programming languages.

### Key Technologies

- **Programming Languages**: C#, Java, JavaScript, TypeScript, Python, Rust
- **Frameworks & SDKs**: 
  - MCP SDK (`@modelcontextprotocol/sdk`)
  - Spring Boot (Java)
  - FastMCP (Python)
  - LangChain4j (Java)
- **Databases**: PostgreSQL with pgvector extension
- **Cloud Platforms**: Azure (Container Apps, OpenAI, Content Safety, Application Insights)
- **Build Tools**: npm, Maven, pip, Cargo
- **Documentation**: Markdown with automated multi-language translation (48+ languages)

### Architecture

- **11 Core Modules (00-11)**: Sequential learning path from fundamentals to advanced topics
- **Hands-on Labs**: Practical exercises with complete solution code in multiple languages
- **Sample Projects**: Working MCP server and client implementations
- **Translation System**: Automated GitHub Actions workflow for multi-language support
- **Image Assets**: Centralized images directory with translated versions

## Setup Commands

This is a documentation-focused repository. Most setup occurs within individual sample projects and labs.

### Repository Setup

```bash
# Clone the repository
git clone https://github.com/microsoft/mcp-for-beginners.git
cd mcp-for-beginners
```

### Working with Sample Projects

Sample projects are located in:
- `03-GettingStarted/samples/` - Language-specific examples
- `03-GettingStarted/01-first-server/solution/` - First server implementations
- `03-GettingStarted/02-client/solution/` - Client implementations
- `11-MCPServerHandsOnLabs/` - Comprehensive database integration labs

Each sample project contains its own setup instructions:

#### TypeScript/JavaScript Projects
```bash
cd <project-directory>
npm install
npm start
```

#### Python Projects
```bash
cd <project-directory>
pip install -r requirements.txt
# or
pip install -e .
python main.py
```

#### Java Projects
```bash
cd <project-directory>
mvn clean install
mvn spring-boot:run
```

## Development Workflow

### Documentation Structure

- **Modules 00-11**: Core curriculum content in sequential order
- **translations/**: Language-specific versions (auto-generated, do not edit directly)
- **translated_images/**: Localized image versions (auto-generated)
- **images/**: Source images and diagrams

### Making Documentation Changes

1. Edit only the English markdown files in the root module directories (00-11)
2. Update images in the `images/` directory if needed
3. The co-op-translator GitHub Action will automatically generate translations
4. Translations are regenerated on push to main branch

### Working with Translations

- **Automated Translation**: GitHub Actions workflow handles all translations
- **Do NOT manually edit** files in `translations/` directory
- Translation metadata is embedded in each translated file
- Supported languages: 48+ languages including Arabic, Chinese, French, German, Hindi, Japanese, Korean, Portuguese, Russian, Spanish, and many more

## Testing Instructions

### Documentation Validation

Since this is primarily a documentation repository, testing focuses on:

1. **Link Validation**: Ensure all internal links work
```bash
# Check for broken markdown links
find . -name "*.md" -type f | xargs grep -n "\[.*\](.*)"
```

2. **Code Sample Validation**: Test that code examples compile/run
```bash
# Navigate to specific sample and run its tests
cd 03-GettingStarted/samples/typescript
npm install && npm test
```

3. **Markdown Linting**: Check formatting consistency
```bash
# Use markdownlint if needed
npx markdownlint-cli2 "**/*.md" "#node_modules"
```

### Sample Project Testing

Each language-specific sample includes its own testing approach:

#### TypeScript/JavaScript
```bash
npm test
npm run build
```

#### Python
```bash
pytest
python -m pytest tests/
```

#### Java
```bash
mvn test
mvn verify
```

## Code Style Guidelines

### Documentation Style

- Use clear, beginner-friendly language
- Include code examples in multiple languages where applicable
- Follow markdown best practices:
  - Use ATX-style headers (`#` syntax)
  - Use fenced code blocks with language identifiers
  - Include descriptive alt text for images
  - Keep line lengths reasonable (no hard limit, but be sensible)

### Code Sample Style

#### TypeScript/JavaScript
- Use ES modules (`import`/`export`)
- Follow TypeScript strict mode conventions
- Include type annotations
- Target ES2022

#### Python
- Follow PEP 8 style guidelines
- Use type hints where appropriate
- Include docstrings for functions and classes
- Use modern Python features (3.8+)

#### Java
- Follow Spring Boot conventions
- Use Java 21 features
- Follow standard Maven project structure
- Include Javadoc comments


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/mcp-for-beginners](https://github.com/microsoft/mcp-for-beginners) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
