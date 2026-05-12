---
trigger: always_on
description: libBIDS.sh is a single-file Bash library (>=4.3) for parsing BIDS (Brain Imaging Data Structure) datasets into TSV format. It provides filtering, iteration, and JSON metadata processing capabilities for neuroimaging data.
---

# Repository Guidelines

## Project Overview

libBIDS.sh is a single-file Bash library (>=4.3) for parsing BIDS (Brain Imaging Data Structure) datasets into TSV format. It provides filtering, iteration, and JSON metadata processing capabilities for neuroimaging data.

**Key characteristics:**
- 794-line Bash library following functional programming patterns
- AWK-based data processing for TSV filtering and column operations
- Extensible custom entity support via JSON configurations
- Zero-dependency core (jq optional for JSON features)

## Architecture & Data Flow

### Single-File Architecture

The entire library lives in `libBIDS.sh` with a plugin architecture:

```
Directory tree → filename parsing → TSV structure → filtering/iteration
                     ↓                    ↓
              glob patterns        AWK processing
              (32 entities)       (column/row ops)
```

### Core Parsing Flow

1. **Pattern Matching**: Uses Bash extended glob patterns to match 32 standard BIDS entities (sub, ses, task, run, etc.)
2. **Filename Parsing**: Regex-based entity extraction into associative arrays
3. **JSON Sidecar Matching**: Exact filename matching (no inheritance resolution)
4. **Output**: TSV format with entity columns, derivatives, data_type, suffix, extension, path

### Public API (7 functions)

- `libBIDSsh_parse_bids_to_table` - Core BIDS parser, main entry point
- `libBIDSsh_table_filter` - AWK-based TSV filtering (columns, rows, drop-na)
- `libBIDSsh_drop_na_columns` - Remove columns with all NA values
- `libBIDSsh_extension_json_rows_to_column_json_path` - JSON metadata column extraction
- `libBIDSsh_table_column_to_array` - Convert TSV column to Bash array
- `libBIDSsh_table_iterator` - Iterate over TSV rows in callbacks
- `libBIDSsh_json_to_associative_array` - Parse JSON into Bash associative array

### Internal Functions (2)

- `_libBIDSsh_parse_filename` - Regex-based entity extraction from filenames
- `_libBIDSsh_load_custom_entities` - Load custom entity definitions from `custom/` directory

## Key Directories

```
libBIDS.sh/
├── libBIDS.sh                    # Main library (all functionality)
├── README.md                     # Comprehensive documentation + API reference
├── generate_entity_patterns.sh   # Utility: generate glob patterns from schema.json
├── custom/                       # Custom entity definitions
│   └── custom_entities.json.tpl # Template for custom entities
└── bids-examples/               # Test datasets (submodule, 40+ datasets)
    ├── run_tests.sh             # BIDS validation script (bids-validator)
    └── default-config.json      # Validator config (ignore EMPTY_FILE warnings)
```

## Development Commands

### Basic Usage

```bash
# Source the library
source libBIDS.sh

# Parse a BIDS dataset to TSV
libBIDSsh_parse_bids_to_table path/to/bids/dataset

# Direct execution (not sourced)
./libBIDS.sh path/to/bids/dataset
```

### Testing

```bash
# Manual testing with example datasets
./libBIDS.sh bids-examples/ds001

# Validate BIDS compliance (requires bids-validator)
cd bids-examples
./run_tests.sh              # Validate all datasets
./run_tests.sh ds001 ds002  # Validate specific datasets
```

### Development Utilities

```bash
# Generate entity patterns from BIDS schema
./generate_entity_patterns.sh  # Requires schema.json and jq
```

## Code Conventions & Common Patterns

### Bash Requirements

- **Bash >= 4.3** required for:
  - Associative arrays (`declare -A`)
  - Namerefs (`local -n`)
  - `readarray` / `mapfile`
- **Strict mode**: `set -euo pipefail`
- **Version check**: Library validates Bash version on load

### Naming Conventions

- **Public functions**: `libBIDSsh_*` prefix (snake_case)
- **Internal functions**: `_libBIDSsh_*` prefix (private)
- **Local variables**: `local` keyword, lowercase with underscores
- **Associative arrays**: Pass by nameref (`local -n arr="$2"`)

### Error Handling

```bash
# Version check with clear error
if ((BASH_VERSINFO[0] < 4 || (BASH_VERSINFO[0] == 4 && BASH_VERSINFO[1] < 3)); then
  echo "Error: bash >= 4.3 is required" >&2
  exit 1
fi

# Directory validation
if [[ ! -d "$bidspath" ]]; then
  echo "Error: Directory '$bidspath' does not exist" >&2
  return 1
fi
```

### Option Parsing Pattern

```bash
# Case statement for options
while [[ $# -gt 0 ]]; do
  case "$1" in
    -c | --columns) columns="$2"; shift 2 ;;
    -r | --row-filter) row_filters+=("$2"); shift 2 ;;
    -d | --drop-na) drop_na_cols="$2"; shift 2 ;;
    -v | --invert) invert_filter="1"; shift ;;
    *) echo "Unknown option: $1" >&2; return 1 ;;
  esac
done
```

### Glob Pattern Usage

```bash
# Enable extended globbing
shopt -s extglob nullglob globstar

# Build BIDS entity patterns
local entities=(
  "*(_sub-+([a-zA-Z0-9]))"
  "*(_ses-+([a-zA-Z0-9]))"
  # ... 30 more entities
)

# Find files
local files=("${bidspath}"/**/${pattern})
```

### AWK Integration

```bash
# TSV processing with AWK
awk -v columns="${columns}" \
    -v row_filters_str="${row_filters_str}" \
    'BEGIN { FS="\t"; OFS="\t" } ...'
```

### JSON Processing

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CoBrALab/libBIDS.sh](https://github.com/CoBrALab/libBIDS.sh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
