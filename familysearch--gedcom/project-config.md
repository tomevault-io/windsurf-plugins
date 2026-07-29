---
trigger: always_on
description: Always reference these instructions first and only fallback to search or bash commands when you encounter unexpected information that does not match the information provided here.
---

# FamilySearch GEDCOM Specification Repository

Always reference these instructions first and only fallback to search or bash commands when you encounter unexpected information that does not match the information provided here.

The FamilySearch GEDCOM repository is a specification repository that defines the standard for genealogical data exchange. This is NOT a software application with runtime components, but rather a documentation project that builds specification documents (HTML/PDF) from markdown source files.

## Working Effectively

### Bootstrap and Build Dependencies
Install all required dependencies in this exact order:
```bash
# Install system dependencies
sudo apt-get update && sudo apt-get install -y pandoc

# Install Python dependencies  
python3 -m pip install --user --upgrade weasyprint mdformat_gfm
```

Verify installations:
```bash
python3 --version        # Should show Python 3.x
pandoc --version         # Should show pandoc version
yamllint --version       # Should show yamllint (pre-installed)
make --version           # Should show GNU Make
```

### Build the Specification
Build the complete specification (HTML and PDF) and extract all generated files:
```bash
cd build
# Ensure tags directory exists (required after distclean)
mkdir -p ../extracted-files/tags
make
```
**NEVER CANCEL: Build takes ~30 seconds. Set timeout to 60+ seconds.**

If the build completes but the `extracted-files/tags/` directory is empty, run the URI extraction manually:
```bash
cd build
python3 extract-yaml.py --spec=../specification/ --dest=../extracted-files/
python3 yaml-to-tsv.py --dest=../extracted-files/ ../extracted-files/tags
```

This command generates:
- `specification/gedcom.html` - Complete specification in HTML format  
- `specification/gedcom.pdf` - Complete specification in PDF format
- `extracted-files/grammar.abnf` - ABNF grammar definitions
- `extracted-files/grammar.gedstruct` - Structure grammar definitions  
- `extracted-files/tags/` - Individual tag definition files
- Various `.tsv` files with cardinalities, enumerations, payloads, substructures

### Clean Generated Files
Remove temporary build files only:
```bash
cd build
make clean
```

Remove all generated files (HTML, PDF, extracted files):
```bash
cd build  
make distclean
# Recreate tags directory for next build
mkdir -p ../extracted-files/tags
```

## Validation

### Validate YAML Files
Always run YAML validation before committing changes:
```bash
yamllint .
```
**Completes in <1 second. This must pass for CI to succeed.**

### Build Validation
Always validate that the build process completes successfully after making changes to specification files:
```bash
cd build
make
```

The build process will show expected warnings about CSS properties (these are normal and documented in build/README.md). Any actual errors will cause the build to fail.

### End-to-End Validation Workflow
After making changes to specification markdown files, always run this complete validation:
```bash
# Validate YAML files
yamllint .

# Clean and rebuild everything
cd build
make distclean
mkdir -p ../extracted-files/tags
make

# If tags directory is empty, run URI extraction manually
python3 extract-yaml.py --spec=../specification/ --dest=../extracted-files/
python3 yaml-to-tsv.py --dest=../extracted-files/ ../extracted-files/tags

# Verify generated files exist
ls -la ../specification/gedcom.html ../specification/gedcom.pdf
ls -la ../extracted-files/grammar.abnf ../extracted-files/grammar.gedstruct
ls -la ../extracted-files/tags/
```

## CI/CD Integration

The repository has automated workflows that run on pushes and pull requests:

### YAML Validation Workflow
- Triggered on: pushes and PRs to main, v7.1, next-patch branches
- Command: `yamllint .`
- Must pass for PRs to be mergeable

### File Generation Workflow  
- Triggered on: pushes to main and v7.1 branches
- Automatically runs grammar and tag extraction
- Creates PRs with updated extracted files if changes detected
- Uses commands:
  - `python3 extract-grammars.py ../specification/gedcom*.md ../extracted-files/`
  - `python3 extract-yaml.py --spec=../specification/ --dest=../extracted-files/`

## Repository Structure

### Key Directories
- `specification/` - Markdown source files for the GEDCOM specification (main content)
- `build/` - Build tools, scripts, and configuration files  
- `extracted-files/` - Auto-generated files (DO NOT EDIT MANUALLY)
- `version-detection/` - Version detection specification
- `.github/workflows/` - CI pipeline definitions

### Specification Source Files (in order)
- `specification/gedcom-0-introduction.md`
- `specification/gedcom-1-hierarchical-container-format.md`  
- `specification/gedcom-2-data-types.md`
- `specification/gedcom-3-structures-1-organization.md`
- `specification/gedcom-3-structures-3-meaning.md`
- `specification/gedcom-3-structures-4-enumerations.md`
- `specification/gedcom-4-gedzip.md`
- `specification/gedcom-5-contributors.md`
- `specification/gedcom-6-appendix-calendars.md`

### Build Tools (build/ directory)
- `Makefile` - Orchestrates the build process
- `hyperlink.py` - Adds hyperlinks to markdown 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FamilySearch/GEDCOM](https://github.com/FamilySearch/GEDCOM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
