---
trigger: always_on
description: Core development rules: APDC methodology, TDD workflow, AI assistant behavior, and code quality
---


# Kubernaut Core Development Rules

## 🚨 **MANDATORY PRINCIPLES**

### 1. Business Requirements Mandate
**EVERY code change MUST be backed by at least ONE business requirement**

**Format**: `BR-[CATEGORY]-[NUMBER]` (e.g., BR-WORKFLOW-001, BR-AI-056)

**Categories**: WORKFLOW, AI, INTEGRATION, SECURITY, PLATFORM, API, STORAGE, MONITORING, SAFETY, PERFORMANCE

**Rules**:
- All tests must map to specific business requirements
- All implementation code must serve documented business needs
- No speculative or "nice to have" code without business backing

---

### 2. Critical Decision Process
**MANDATORY**: Ask for input on ALL critical decisions:
- Architecture changes and design patterns
- New dependencies or external integrations
- Performance trade-offs and optimization decisions
- Security implementations and access controls
- Refactoring that affects system complexity

**Format**: Provide recommendation with detailed justification when asking

---

### 3. APDC Methodology (Complex Tasks)
**Use for**: Complex features, refactoring, new components, build error fixing, AI/ML development

**Phases**:
1. **Analysis** (5-15 min): Context + business alignment + risk assessment
2. **Plan** (10-20 min): Strategy + TDD mapping + **user approval required**
3. **Do** (Variable): RED → GREEN → REFACTOR with validation checkpoints
4. **Check** (5-10 min): Validation + confidence assessment (60-100%)

**See**: [Complete APDC Framework](mdc:docs/development/methodology/APDC_FRAMEWORK.md)
**Quick Ref**: [APDC Quick Reference](mdc:docs/development/methodology/APDC_QUICK_REFERENCE.md)

---

### 4. TDD Workflow (All Development)
**MANDATORY**: RED → GREEN → REFACTOR (tests first, always)

1. **RED**: Write failing tests defining business contract
2. **GREEN**: Minimal implementation + MANDATORY main app integration
3. **REFACTOR**: Enhance existing code with sophisticated logic

**NEVER**: Use `Skip()` to avoid test failures
**NEVER**: Skip REFACTOR phase

---

## 🤖 **AI ASSISTANT BEHAVIOR - MANDATORY CHECKPOINTS**

### **CHECKPOINT A: Type Reference Validation**
**TRIGGER**: About to reference any struct field (e.g., `object.FieldName`)

**MANDATORY ACTION**:
```bash
# HALT: Read type definition file BEFORE referencing fields
read_file [type_definition_file]
# RULE: Verify field exists in struct definition
```

**Violation**: "🚨 Type reference attempted without validation - DEVELOPMENT STOPPED"

---

### **CHECKPOINT B: Test Creation Validation**
**TRIGGER**: About to create test file with business logic references

**MANDATORY ACTION**:
```bash
# HALT: Search for existing implementations FIRST
codebase_search "existing [ComponentType] implementations"
grep -r "[ComponentType]" pkg/ --include="*.go"
# RULE: Enhance existing patterns instead of creating new
```

**Violation**: "🚨 Test creation attempted without existing implementation analysis - DEVELOPMENT STOPPED"

---

### **CHECKPOINT C: Business Integration Validation**
**TRIGGER**: Creating new business types or interfaces

**MANDATORY ACTION**:
```bash
# HALT: Verify main application integration
grep -r "[NewComponentType]" cmd/ --include="*.go"
# RULE: Business code MUST be integrated in main applications (cmd/)
```

**Violation**: "🚨 Business component creation attempted without main app integration validation - DEVELOPMENT STOPPED"

---

### **CHECKPOINT D: Build Error Investigation**
**TRIGGER**: User reports build errors or undefined symbols

**MANDATORY ACTION**:
```bash
# HALT: Execute comprehensive symbol analysis
codebase_search "[undefined_symbol] usage patterns and dependencies"
grep -r "[undefined_symbol]" . --include="*.go" -n
go build [affected_file] 2>&1
# RULE: Present complete analysis with options A/B/C before implementation
```

**Required Report Format**:
```
🚨 UNDEFINED SYMBOL ANALYSIS:
Symbol: [undefined_symbol]
References found: [N files with paths]
Dependent infrastructure: [list missing types/functions]
Scope: [minimal/medium/extensive with evidence]

OPTIONS (Evidence-Based):
A) Implement complete infrastructure ([X] files affected)
B) Create minimal stub ([Z] files affected, may break [W] files)
C) Alternative approach: [evidence-based alternative]

🚫 MANDATORY USER DECISION REQUIRED: Which approach? (A/B/C)
```

**Violation**: "🚨 Build error resolution attempted without comprehensive analysis + user approval - DEVELOPMENT STOPPED"

---

## 🚫 **FORBIDDEN AI ACTIONS**

**NEVER DO THESE**:
1. **NEVER** reference struct fields without first reading the type definition file
2. **NEVER** assume testutil types exist - always validate with `read_file` or `grep`
3. **NEVER** create test code without first using `codebase_search` for existing implementations
4. **NEVER** generate business types without confirming main application usage
5. **NEVER** proceed if any validation step fails
6. **NEVER** implement missing types without full dependency analysis (CHECKPOINT D)

---

## 💻 **CODE QUALITY STANDARDS**

### Error Handling (MANDATORY)
- **ALWAYS** handle errors, never ignore them
- **ALWAYS** add log entry for every error
- Use structured error types from `internal/errors/`
- Wrap errors with context: `fmt.Errorf("description: %w", err)`

### Type System

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordigilh/kubernaut](https://github.com/jordigilh/kubernaut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
