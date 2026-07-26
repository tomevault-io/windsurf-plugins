---
trigger: always_on
description: Pyteomics is a proteomics data analysis library focused on parsing mass spectrometry file formats and peptide/protein analysis. The architecture centers around format-specific parsers, including those built on a common XML parsing framework, as well as other specialized parsers for non-XML formats.
---

# Pyteomics Copilot Instructions

## Project Overview
Pyteomics is a proteomics data analysis library focused on parsing mass spectrometry file formats and peptide/protein analysis. The architecture centers around format-specific parsers, including those built on a common XML parsing framework, as well as other specialized parsers for non-XML formats.

## Project Structure
The project follows a standard Python package layout:

### Main Code (`pyteomics/`)
- **Format parsers**: Individual modules for each supported format (e.g., `mzml.py`, `pepxml.py`, `mgf.py`, `fasta.py`)
- **Core modules**: `xml.py` (XML parsing framework), `parser.py` (peptide sequences), `proforma.py` (modern sequence notation)
- **Specialized subpackages**:
  - `auxiliary/`: Core utilities, base classes, and helper functions (`file_helpers.py`, `structures.py`, `utils.py`, `target_decoy.py`)
  - `mass/`: Mass calculations and UniMod integration (`mass.py`, `unimod.py`)
  - `openms/`: OpenMS-specific formats (`featurexml.py`, `idxml.py`, `trafoxml.py`)
- **Analysis modules**: `achrom.py` (chromatography), `electrochem.py` (electrochemistry), `pylab_aux.py` (plotting utilities)

### Tests (`tests/`)
- Testing framework: `unittest` with individual test files for each module. Tests should be run from the `tests/` directory to ensure correct imports.
- Test files: `test_*.py` for each module with corresponding test data files (`test.mzML`, `test.pepxml`, etc.)
- Controlled vocabularies: OBO files for CV validation (`psi-ms.obo`, `PSI-MOD.obo`)
- Test databases: `unimod.db` and `unimod.xml.gz` for UniMod testing

### Documentation (`doc/`)
- **Source**: RST files organized by topic (`data.rst`, `parser.rst`, `mass.rst`, etc.)
- **API reference**: Comprehensive module documentation in `source/api/`
- **Examples**: Practical usage examples in `source/examples/`
- **Build system**: Sphinx configuration and Makefile for HTML generation

## Core Architecture

### Unified Parser Hierarchy
All parsers inherit from common base classes in `pyteomics/auxiliary/file_helpers.py`:
- `FileReader`: Base class for all format parsers (XML and non-XML)
- `IndexedReaderMixin`: Adds indexing capabilities for random access
- `TaskMappingMixin`: Enables parallel processing via `map()` method

### XML Parser Hierarchy
XML-based format parsers additionally inherit from `pyteomics/xml.py`:
- `XML`: Base iterative parser with schema support
- `IndexedXML`: Adds byte-offset indexing for random access
- `MultiProcessingXML`: Combines indexing with parallel processing
- `IndexSavingXML`: Persistent index caching to disk

**XML Pattern**: Each XML format (mzML, pepXML, mzIdentML, etc.) combines some of these mixins:
```python
class MzML(BinaryArrayConversionMixin, CVParamParser, TimeOrderedIndexedReaderMixin, MultiProcessingXML, IndexSavingXML):
```

### Non-XML Parser Hierarchy
Non-XML formats (MGF, FASTA, MS1/MS2) use the same base infrastructure:
```python
class MGF(MGFBase, FileReader)                    # Sequential parser
class IndexedMGF(MGFBase, TaskMappingMixin, TimeOrderedIndexedReaderMixin, IndexSavingTextReader)  # Indexed parser
```

### Common Parser Interface
Every format parser provides:
- **Class-based**: `FormatClass(file, **kwargs)` - full-featured parser (iterative or indexed)
- **Functional**: `read(file, **kwargs)` - old functional interface, returns an instance of the parser class
- **Chain**: `chain(files, **kwargs)` - multi-file processing
- **Indexed access**: `parser[element_id]` when `use_index=True` or when directly instantiating an indexed parser

**Critical Parameters**:
- `iterative=True`: Memory-efficient streaming (default)
- `use_index=False`: Enable random access indexing
- `read_schema=False`: Parse XSD schemas for type conversion
- `retrieve_refs=True`: Auto-resolve ID references (mzIdentML)

## Key Modules

### `pyteomics/auxiliary/`
Core utilities and base classes:
- `file_helpers.py`: `FileReader`, indexing mixins, multiprocessing
- `structures.py`: `BasicComposition`, charge handling, unit types
- `utils.py`: Binary array conversion, base64 decoding
- `target_decoy.py`: FDR calculation algorithms

### `pyteomics/mass/`
- `mass.py`: Amino acid masses, isotope calculations
- `unimod.py`: UniMod database integration (requires SQLAlchemy)

### `pyteomics/parser.py`
Peptide sequence parsing using "modX" notation:
- `parse()`: "H-PEPTIDE-OH" → structured format
- `cleave()`: Enzymatic digestion simulation
- `amino_acid_composition()`: AA counting

### `pyteomics/proforma.py`
ProForma peptide sequence notation support:
- Modern standard for modified peptide sequences
- Controlled vocabulary integration
- Cross-conversion with modX format via `to_proforma()`

**Two Peptide Sequence Formats**:
- **modX**: Legacy format (e.g., "H-oxMPEPTIDE-OH") - simple, readable
- **ProForma**: Modern standard (e.g., "[Oxidation]M-PEPTIDE") - CV-based, precise

## Testing Strategy

### Test Structure
- Individual test files: `tests/test_*.py` for each module

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [levitsky/pyteomics](https://github.com/levitsky/pyteomics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
