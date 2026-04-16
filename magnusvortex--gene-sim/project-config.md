---
trigger: always_on
description: A genealogical simulation system that models populations of creatures with genetic traits across multiple generations. The system simulates breeding strategies, genetic inheritance, mutations, and provides comprehensive post-simulation reporting and analysis.
---

# Genealogical Simulation Project - Cursor Rules

## Project Overview
A genealogical simulation system that models populations of creatures with genetic traits across multiple generations. The system simulates breeding strategies, genetic inheritance, mutations, and provides comprehensive post-simulation reporting and analysis.

## Core Domain Entities
- **Creatures**: Entities with diploid genomes, genotypes, and phenotypes
- **Traits**: Genetic characteristics supporting multiple inheritance patterns (simple Mendelian, incomplete dominance, codominance, sex-linked, polygenic)
- **Breeders**: Pluggable breeding strategies (random, selective, inbreeding avoidance; fitness-based in Phase 2)
- **Mutations**: Point mutations with configurable rates, tracked through generations with complete history (Phase 2)
- **Populations**: Groups of creatures evolving over generations with demographic tracking
- **Generations**: Time-series snapshots of population state
- **Simulations**: Complete experimental runs with configuration and results
- **Reporting**: Post-simulation charts, graphs, and data exports (CSV/JSON)

## Key Design Principles

### 1. Genetic Accuracy
- Implement Mendelian inheritance with expected ratios (3:1, 9:3:3:1, etc.)
- Maintain Hardy-Weinberg equilibrium under random mating with no selection
- Support sex-linked traits with correct inheritance patterns
- Validate mutation rates match configured parameters (Phase 2)

### 2. Extensibility
- Use strategy pattern for breeding algorithms (pluggable and testable)
- Define traits as data (YAML/JSON configuration), not code
- Support custom trait types through configuration
- Clear interfaces for extending functionality

### 3. Reproducibility
- All randomness must use seeded pRNG (NumPy random with explicit seed)
- Store pRNG seed with each simulation run for exact reproduction
- Same seed + configuration = identical results
- Deterministic behavior is critical for scientific validity

### 4. Performance Requirements
- Small sim (100 creatures, 50 gen): < 5 seconds
- Medium sim (1,000 creatures, 100 gen): < 60 seconds
- Large sim (10,000 creatures, 100 gen): < 10 minutes
- Memory usage (1,000 creatures): < 500 MB
- Use NumPy arrays for genetic operations
- Batch database operations where possible
- Pre-aggregate statistics during simulation

### 5. Data Integrity
- Complete lineage tracking (forward and backward traversal)
- All mutation events individually traceable (Phase 2)
- SQLite foreign keys enforce referential integrity
- ACID compliance ensures data consistency
- No data loss across generations

### 6. Reporting-First Architecture
- Design for post-simulation analysis, not real-time visualization
- Store all data in SQLite for efficient querying
- Support time-series queries (trait frequencies by generation)
- Enable lineage queries without loading entire simulation
- Optimize indexes for common reporting patterns

## Technology Stack
- **Language**: Python 3.10+
- **Database**: SQLite (built-in, no external server)
- **Numerical**: NumPy for efficient arrays and seeded random number generation
- **Visualization**: Matplotlib/Plotly (optional, for reporting)
- **Data Export**: Pandas for CSV/JSON and SQLite querying
- **Configuration**: YAML/JSON parsing
- **Testing**: pytest with >80% coverage target

## Code Style & Conventions

### Python Standards
- Follow PEP 8 style guidelines
- Use type hints for all function signatures and class attributes
- Prefer dataclasses or Pydantic models for data structures
- Use enums for trait types and other fixed sets of values
- Document all public APIs with docstrings (Google style)

### File Organization
- Domain models in `models/` package (creature, trait, breeder, etc.)
- Simulation engine in `simulation/` package
- Database layer in `database/` package
- Reporting/analysis in `reporting/` package
- Configuration loading in `config/` package
- Tests mirror source structure in `tests/`

### Naming Conventions
- Classes: PascalCase (e.g., `Creature`, `Trait`, `RandomBreeder`)
- Functions/methods: snake_case (e.g., `select_mates`, `is_eligible_for_breeding`; `calculate_fitness` in Phase 2)
- Constants: UPPER_SNAKE_CASE (e.g., `MAX_TRAIT_ID`, `DEFAULT_MUTATION_RATE`)
- Database tables: plural snake_case (e.g., `creatures`, `traits`, `genotypes`)

### Error Handling
- Use custom exception classes for domain-specific errors
- Validate inputs at boundaries (configuration loading, API entry points)
- Fail fast with clear error messages
- Log errors with sufficient context for debugging

## Database Design

### SQLite Schema Principles
- Use foreign keys with ON DELETE CASCADE where appropriate
- Create indexes for common query patterns (birth_generation, trait_id, lineage)
- Store pRNG seed with each simulation for reproducibility
- Normalize where it improves query performance
- Use CHECK constraints for data validation (trait_id range 0-99, etc.)

### Query Optimization
- Prefer indexed columns in WHERE clauses
- Use batch inserts for generation data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MagnusVortex) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
