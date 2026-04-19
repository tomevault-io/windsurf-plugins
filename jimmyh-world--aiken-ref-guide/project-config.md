---
trigger: always_on
description: Unified development rules for the Aiken Reference Guide repository - examples and documentation
---


# Unified Aiken Reference Guide Development Rules

You are an expert developer working on the **Aiken Developer's Reference Guide repository**. Your role encompasses both creating production-grade examples and maintaining accurate, battle-tested documentation based on real implementation experience.

## Repository Mission

This repository serves as the **definitive knowledge base** for Aiken smart contract development, designed for both human developers and AI assistants. Everything you create must meet production standards while serving as educational resources.

## Core Development Philosophy

### 1. Reference-Guide-First Development

**ALWAYS Use the Reference Guide**:

- **Start Here**: Review `docs/` before writing any code
- **Pattern Selection**: Use `docs/patterns/` to identify applicable architectural approaches
- **Security Implementation**: Follow `docs/security/` guidelines religiously
- **Performance Optimization**: Apply techniques from `docs/performance/`
- **Integration Design**: Reference `docs/integration/` for off-chain compatibility

**Documentation Validation Loop**:

```
1. Reference docs → 2. Implement example → 3. Note gaps/errors → 4. Update docs → 5. Improve examples
```

### 2. Production-Grade Standards

**"Deploy Tomorrow" Principle**: Every example must be ready for mainnet deployment with proper testing and security audits.

**Quality Requirements**:

- Complete error handling and edge case coverage
- Comprehensive test suites (>95% coverage)
- Security audit checklist completion
- Performance benchmarking and optimization
- Professional documentation with tutorials

### 3. Living Documentation Maintenance

**Experience-Driven Updates**: Documentation must evolve based on actual development experiences and real-world usage.

**Continuous Improvement**: When you discover gaps, outdated information, or better approaches while developing examples, immediately note them for documentation updates.

## Development Workflow

### Phase 1: Reference Guide Analysis

Before writing any code:

```
1. **Documentation Review**:
   - Read relevant sections in docs/patterns/ for architectural guidance
   - Review docs/security/ for applicable threats and mitigations
   - Check docs/performance/ for optimization techniques
   - Study docs/integration/ for off-chain compatibility requirements
   - Examine existing examples/ for consistency patterns

2. **Gap Identification**:
   - Note any missing information in docs/
   - Identify outdated or incorrect guidance
   - Record areas where documentation is unclear
   - Document concepts that need better examples

3. **Implementation Planning**:
   - Choose patterns from docs/patterns/ that apply
   - Design security model using docs/security/ guidelines
   - Plan testing strategy referencing docs/language/testing.md
   - Design integration approach using docs/integration/ guides
```

### Phase 2: Production-Grade Implementation

```
**Code Development Standards**:
- Follow validator patterns from docs/language/validators.md
- Implement security measures from docs/security/validator-risks.md
- Use custom types as specified in docs/language/data-structures.md
- Apply CEI pattern and avoid anti-patterns from docs/security/anti-patterns.md
- Optimize using techniques from docs/performance/optimization.md

**Testing Excellence**:
- Unit tests for all functions (reference docs/language/testing.md)
- Property-based tests for invariants
- Performance benchmarks using docs/performance/benchmarking.md
- Integration tests with off-chain tools from docs/integration/
- Security test scenarios based on docs/security/validator-risks.md

**Documentation Requirements**:
- Comprehensive README following examples/ pattern
- Inline code documentation explaining WHY decisions were made
- Architecture explanations linking to relevant docs/ sections
- Security rationale referencing docs/security/ guidelines
- Integration examples using docs/integration/ tools
```

### Phase 3: Documentation Enhancement

```
**Learning Extraction**:
Track discoveries while developing:
- New patterns that emerged during implementation
- Security vulnerabilities discovered and how they were mitigated
- Performance optimizations that had significant impact
- Integration challenges and solutions found
- Documentation gaps or inaccuracies encountered

**Documentation Updates Required**:
- Add new patterns to docs/patterns/ if discovered
- Update security documentation in docs/security/ with real vulnerabilities found
- Enhance performance guides in docs/performance/ with measured improvements
- Improve integration guides in docs/integration/ with working solutions
- Fix any incorrect information discovered during implementation
- Add cross-references between examples and documentation

**Quality Validation**:
- Verify all documentation updates are accurate and tested
- Ensure examples still work after documentation changes
- Update cross-references throughout the repository
- Validate that new documentation can be followed by others
```

## Implementation Standards

### Security-First Development

**Reference docs/security/ extensively**:

- Implement threat mitigations from docs/security/validator-risks.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jimmyh-world) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
