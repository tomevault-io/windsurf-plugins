---
trigger: always_on
description: 1. [File Types and Formats](#file-types-and-formats)
---

# Fortran Coding Standards

## Table of Contents
1. [File Types and Formats](#file-types-and-formats)
2. [Naming Conventions](#naming-conventions)
3. [Coding Rules](#coding-rules)
4. [Tooling and Navigation](#tooling-and-navigation)
5. [Template Structure](#template-structure)
6. [Code Examples](#code-examples)
7. [Common Pitfalls](#common-pitfalls)

## File Types and Formats

### Legacy Files (.F)
- Fixed-form format with 132 character line limit
- Uppercase file extension required
- Use only for maintaining existing legacy code

### New Files (.F90)
- Free-form format (recommended for all new development)
- Modern Fortran 90+ features available
- More flexible syntax and readability

## Naming Conventions

### File Names
- **Subroutines**: `<subroutine_name>.F90`
- **Modules**: `<module_name>_mod.F90`
- Use lowercase with underscores for separation

### Variable and Procedure Names
- Use descriptive, clear names
- Separate words with underscores
- Constants in UPPERCASE
- Module names in lowercase with `_mod` suffix

## Coding Rules

| **DO** | **DON'T** |
|--------|-----------|
| Use Fortran 90+ features | Runtime polymorphism, type-bound procedures |
| Use `*.F` (fixed, 132 chars) for legacy files | Use tabs for indentation |
| Indent using 2 spaces | Use `COMMON`, `EQUIVALENCE`, `SAVE` |
| Use modules and derived types | Use global variables |
| Pass variables as dummy arguments | Use `GOTO`, multiple `RETURN` statements |
| Look for clarity in code | Use assumed-size arrays `A(*)` |
| Explicit array sizes: `INTEGER, INTENT(IN) :: A(LEN)` | Use `DOUBLE PRECISION` systematically |
| Use bounds for array operations | Perform `A = B + C` without bounds checking |
| Use the `MY_REAL` type for real numbers | Use pointers when allocatable is possible |
| Use `ALLOCATABLE` arrays | Use large automatic arrays |
| Use `MY_ALLOC` and check allocation status | Rely on automatic deallocation |
| Deallocate arrays as soon as possible | Leave arrays allocated unnecessarily |

### Additional Rules
- **Line Length**: Keep lines under 120 characters for readability
- **Comments**: Use `!` for inline comments, `!!` for documentation
- **Intent Declarations**: Always specify `INTENT(IN)`, `INTENT(OUT)`, or `INTENT(INOUT)`
- **Routine Length**: 
  - Leaf routines: ≤ 200 lines
  - Main routines: ≤ 1000 lines
- **DO NOT USE POINTER** when this can be avoided. In particular pointers to noncontiguous data are forbidden because of performance issues.

## Tooling and Navigation

- Prefer the OpenRadioss MCP index tools (`openradioss-index-*`) for code navigation and symbol lookup instead of grep/rg whenever possible.

## Template Structure

### Standard .F90 File Template

```fortran
module my_subroutine_mod
  implicit none
contains

! ======================================================================================================================
!                                                   PROCEDURES
! ======================================================================================================================

!! \brief Brief description of the routine
!! \details More detailed description if needed, including:
!!          - Purpose and functionality  
!!          - Input/output parameter descriptions
!!          - Any special considerations or limitations
subroutine subroutine_example(intbuf_tab, buffer, buffer_size, acceleration, acceleration_size)

! ----------------------------------------------------------------------------------------------------------------------
!                                                   MODULES
! ----------------------------------------------------------------------------------------------------------------------
! Module names must be in uppercase (will change later)
! ONLY is mandatory, note the space before the comma
  use INTBUF_DEF_MOD, only : intbuf_struct
  use CONSTANT_MOD, only : PI
  use PRECISION_MOD, only : WP
  use MVSIZ_MOD, only : MVSIZ
  use NAMES_AND_TITLES_MOD, only : ncharline100
  use MY_ALLOC_MOD, only : my_alloc

! ----------------------------------------------------------------------------------------------------------------------
!                                                   IMPLICIT NONE
! ----------------------------------------------------------------------------------------------------------------------
  implicit none

! ----------------------------------------------------------------------------------------------------------------------
!                                                   INCLUDED FILES
! ----------------------------------------------------------------------------------------------------------------------
! No comments on the same line as #include, #define, #ifdef, #endif
! Generally speaking, #include is forbidden with few exceptions

! ----------------------------------------------------------------------------------------------------------------------
!                                                   ARGUMENTS
! ----------------------------------------------------------------------------------------------------------------------
  type(intbuf_struct), intent(in)    :: intbuf_tab                    !< Input buffer structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenCourant/OpenCourant](https://github.com/OpenCourant/OpenCourant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
