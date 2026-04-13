---
trigger: always_on
description: 1. **CONSULTA OBLIGATORIA DEL HANDBOOK**: Antes de cualquier acción, el asistente DEBE leer `.budgetpro/handbook/AXIOM_SAFE_OPERATIONS.md`. Es el oráculo de decisiones para MODES, RISK y NAMING.
---

# BudgetPro Governing Rules for AI and Development

#

# This file defines the mandatory guidelines that govern how the codebase

# must be modified and how AI assistants should operate when interacting

# with the repository. It complements the AXIOM configuration and reflects

# recent hardening of the Naming, Boundary, State Machine and Semgrep

# validators. All changes and suggestions must comply with the rules

# defined here.

## ⚖️ LEY SUPREMA DE GOBERNANZA (AXIOM-SAFE)

1. **CONSULTA OBLIGATORIA DEL HANDBOOK**: Antes de cualquier acción, el asistente DEBE leer `.budgetpro/handbook/AXIOM_SAFE_OPERATIONS.md`. Es el oráculo de decisiones para MODES, RISK y NAMING.

2. **DIAGNÓSTICO DE APERTURA (MANDATORIO)**:
   - No se permite código sin antes imprimir en el chat el **DIAGNÓSTICO INICIAL**:
     ```
     📊 [MODO: 0/1/2] | [RIESGO: LOW/MID/HIGH] | [BRANCH: tipo/sistema-desc]
     Estado: [Resumen corto del repo según ./axiom.sh --status]
     ```

3. **JERARQUÍA DE ALERTAS**:
   - **[BLOCKING]**: Detención absoluta. Prohibido el uso de `--no-verify` a menos que se documente un incidente en `.budgetpro/handbook/INCIDENT_LOGS.md` bajo MODE_0.
   - **[WARNING]**: Los avisos de 'LEAK' o 'SECRET' se tratan como BLOCKING. Los de 'Null-Safety' deben corregirse en el batch actual.

4. **ESTÁNDAR DE TRABAJO ATÓMICO**:
   - **1 Propósito = 1 Commit**.
   - Si un cambio excede el **Blast Radius (10 archivos)**, el asistente DEBE realizar un `git reset` y dividir el trabajo en Batches Temáticos (ej: batch_logic, batch_tests).

5. **LIMPIEZA DE CÓDIGO (ANTI-DIRTY)**:
   - Prohibido eliminar `System.out` o `printStackTrace` sin reemplazarlos por una implementación formal de `Logger` (SLF4J/java.util.logging).
   - Todo método público nuevo o modificado DEBE incluir validación `Objects.requireNonNull()`.

6. **PROTECCIÓN DE ARCHIVOS DE GOBIERNO**:
   - El asistente tiene PROHIBIDO modificar `.cursorrules.md` o `axiom.config.yaml` sin autorización explícita.
   - Cualquier archivo llamado `.cursorrules` (sin extensión) es considerado CORRUPTO y debe ser purgado según SOP-02.

supreme_rule:

# AXIOM is the ultimate authority. The domain layer is sacrosanct and

# must remain free of dependencies on frameworks or upper layers.

- "AXIOM IS LAW: all code changes must pass `./axiom.sh --dry-run` before execution and respect the hexagonal architecture."
- "Domain layer is sacrosanct: ZERO framework dependencies (Spring, Hibernate, JPA, Jakarta, etc.)."
- "Domain can ONLY import: java.*, java.util.*, java.math.*, java.time.* (Java Standard Library only)."
- "Any import of org.springframework.*, jakarta.*, javax.*, or any framework package in com.budgetpro.domain is a BLOCKING violation."
- "Never modify the domain layer (`com.budgetpro.domain`) to fix errors in the application or infrastructure layers."
- "Canonical Notebooks are the Source of Truth: NOTEBOOK > CODE (per AI_AGENT_PROTOCOL.md, Rule 3)."

hands_clean:

# Protocol for handling large compilation failures (>50 errors).

- "Extract only the first 20 errors to identify the root cause before editing any files."
- "Prioritise correcting imports or calls in Application/Infrastructure; do not change Domain to satisfy upper-layer errors."
- "If the root cause truly resides in Domain, stop and create an `implementation_plan.md` for approval before proceeding."
- "After each individual change, run `./axiom.sh --dry-run` to validate before making the next change."

initialization:

- "Before writing any code, always run `./axiom.sh --dry-run` to validate the current state."
- "Verify that there are no pending AXIOM violations."
- "Consult `.budgetpro/axiom.config.yaml` and this file (`.cursorrules.md`) to understand the active rules."

architecture:

# Hexagonal architecture enforcement.

- "Domain ← Application ← Infrastructure: dependencies flow inward."
- "Domain depends on nothing; Application depends only on Domain; Infrastructure depends on both."
- "Use Naming, Boundary, State Machine and Semgrep validators to verify adherence to these principles."

blast_radius:

# Limits on the number of files modified per commit in sensitive areas.

red: - "domain/finanzas/presupuesto: max 1 file per commit" - "domain/finanzas/estimacion: max 1 file per commit" - "domain/valueobjects: max 1 file per commit" - "domain/entities: max 1 file per commit"
yellow: - "infrastructure/persistence: max 3 files per commit"
green: - "application: max 10 files per commit" - "infrastructure/web: max 10 files per commit" - "tests: max 10 files per commit"

ai_assistant:

# Mandatory checks for AI assistants when preparing changes.

- "Check that the blast radius limits are respected for the proposed change."
- "Confirm that no secrets or sensitive data are exposed."
- "Ensure no lazy code patterns remain (e.g., methods returning null without validation)."
- "Verify adherence to the hexagonal architecture and domain isolation."
- "Run Naming, Boundary, State Machine and Semgrep validators as part of the validation pipeline."
- "Classify each proposed change as LOW, MID or HIGH risk and ensure the appropriate operating mode is engaged."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/carleetho) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
