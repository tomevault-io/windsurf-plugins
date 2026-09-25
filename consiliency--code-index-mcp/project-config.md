---
trigger: always_on
description: - **Build**: `make test` (builds and tests the plugin)
---

# HTML/CSS Plugin Agent Configuration

## ESSENTIAL_COMMANDS
- **Build**: `make test` (builds and tests the plugin)
- **Test**: `pytest tests/test_html_css_plugin.py -v` (run HTML/CSS plugin tests)
- **Lint**: `make lint` (check code style)
- **Install**: `pip install -e .` (install in development mode)
- **Plugin Test**: `python -m pytest mcp_server/plugins/html_css_plugin/ -v`
- **Tree-sitter**: `pip install tree-sitter-languages` (HTML/CSS grammar support)
- **CSS Validation**: Use online validators for test fixture validation

## CODE_STYLE_PREFERENCES
- **Functions**: snake_case (`extract_html_elements`, `parse_css_selectors`)
- **Classes**: PascalCase (`HtmlCssParser`, `SelectorExtractor`)
- **Files**: snake_case (`html_css_plugin.py`, `test_css_parser.py`)
- **Constants**: UPPER_SNAKE_CASE (`HTML_EXTENSIONS`, `CSS_PROPERTIES`)
- **Variables**: snake_case (`element_tree`, `selector_map`)

## ARCHITECTURAL_PATTERNS
- **Plugin Pattern**: Inherit from `IPlugin` base class with standardized interface
- **Dual Parser Pattern**: Separate Tree-sitter parsers for HTML and CSS
- **Cross-Reference Tracking**: Map CSS selectors to HTML elements
- **Symbol Storage**: Store symbols via `SQLiteStore` with relationship tracking
- **Error Handling**: Return `Result[T, Error]` pattern for all operations
- **Preprocessor Support**: Handle SCSS, SASS, LESS with appropriate parsers

## DEVELOPMENT_ENVIRONMENT
- **Python Version**: 3.8+ (required for Tree-sitter)
- **Dependencies**: `tree-sitter>=0.20.0`, `tree-sitter-languages>=1.8.0`
- **Virtual Environment**: Required (`python -m venv venv`)
- **Test Files**: Place HTML/CSS samples in `tests/fixtures/html_css/`
- **Browser Testing**: Optional validation with headless browser
- **IDE Setup**: Configure for HTML/CSS syntax highlighting

## NAMING_CONVENTIONS
- **Symbol Types**: Use web-specific types (`element`, `selector`, `property`, `rule`)
- **File Extensions**: Support `.html`, `.htm`, `.css`, `.scss`, `.sass`, `.less`
- **Test Files**: `test_html_*.py`, `test_css_*.py` pattern for plugin tests
- **Fixtures**: `html_sample_*.html`, `css_sample_*.css` for test files
- **Selectors**: Track class, id, tag, attribute, pseudo selectors

## TEAM_SHARED_PRACTICES
- **Testing**: Include responsive design, CSS Grid, Flexbox patterns
- **Documentation**: Include HTML/CSS code examples in docstrings
- **Cross-referencing**: Track relationships between HTML elements and CSS rules
- **Error Messages**: Include line numbers and CSS property context
- **Performance**: Target <100ms for parsing, handle large CSS frameworks

## Implementation Status
✅ **FULLY IMPLEMENTED** - Complete HTML/CSS plugin with cross-referencing capabilities

## Overview
The HTML/CSS plugin will provide code intelligence for HTML (.html, .htm) and CSS (.css, .scss, .sass, .less) files using Tree-sitter for parsing. It will track DOM structure, CSS selectors, style definitions, and the relationships between HTML elements and their styles.

## Implementation Requirements

### 1. Tree-sitter Integration
**Grammars**: Use `tree-sitter-html` and `tree-sitter-css` from tree-sitter-languages
```python
from ...utils.treesitter_wrapper import TreeSitterWrapper
from ...utils.fuzzy_indexer import FuzzyIndexer
from ...storage.sqlite_store import SQLiteStore

class Plugin(IPlugin):
    lang = "html_css"
    
    def __init__(self, sqlite_store: Optional[SQLiteStore] = None) -> None:
        self._ts = TreeSitterWrapper()
        self._indexer = FuzzyIndexer(sqlite_store=sqlite_store)
        self._sqlite_store = sqlite_store
        self._repository_id = None
        
        # Separate parsers for HTML and CSS
        self._html_parser = None
        self._css_parser = None
        
        # Track relationships between HTML and CSS
        self._style_index = {}  # selector -> styles mapping
        self._id_index = {}     # id -> elements mapping
        self._class_index = {}  # class -> elements mapping
        
        self._preindex()
```

### 2. File Support Implementation
```python
def supports(self, path: str | Path) -> bool:
    """Support HTML and CSS files including preprocessors"""
    html_suffixes = {'.html', '.htm', '.xhtml'}
    css_suffixes = {'.css', '.scss', '.sass', '.less', '.styl'}
    suffix = Path(path).suffix.lower()
    return suffix in html_suffixes or suffix in css_suffixes

def _get_file_type(self, path: Path) -> str:
    """Determine if file is HTML or CSS"""
    suffix = path.suffix.lower()
    if suffix in {'.html', '.htm', '.xhtml'}:
        return 'html'
    elif suffix in {'.css', '.scss', '.sass', '.less', '.styl'}:
        return 'css'
    return 'unknown'
```

### 3. HTML Symbol Extraction
**Target Node Types** (from Tree-sitter HTML grammar):
- `element` - HTML elements
- `start_tag` - Opening tags
- `end_tag` - Closing tags
- `attribute` - Element attributes
- `attribute_name` - Attribute names
- `attribute_value` - Attribute values
- `text` - Text content
- `script_element` - Script tags
- `style_element` - Style tags

**Implementation Strategy**:
```python
def _index_html_file(self, path: Path, content: str) -> dict:
    """Parse and index HTML file"""
    tree = self._html_parser.parse(content.encode('utf-8'))
    root = tree.root_node
    
    symbols = []

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Consiliency/Code-Index-MCP](https://github.com/Consiliency/Code-Index-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
