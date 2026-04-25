---
trigger: always_on
description: This file contains important development context and reminders for Claude when working on the SegnoMMS project.
---

# Claude Development Context

This file contains important development context and reminders for Claude when working on the SegnoMMS project.


## ✅ Pydantic v2 + MyPy Modernization Achievement

**🎉 COMPLETED:** SegnoMMS configuration system successfully modernized to Pydantic v2 + MyPy strict typing patterns.

### Achievement Summary (as of 2025-01-25)

**Core Modernization Completed:**
- ✅ **Removed `use_enum_values=True`** - Config models now return enum objects at runtime
- ✅ **100% Config Module MyPy Compliance** - All 9 config modules pass strict MyPy validation
- ✅ **Discriminated Unions** - Shape configurations use proper type narrowing with `Field(discriminator="shape")`
- ✅ **TypedDict Patterns** - Type-safe **kwargs for shape renderers using `Unpack[TypedDict]`
- ✅ **Modern Typing Practices** - `from __future__ import annotations`, strict MyPy settings
- ✅ **Enum Objects at Runtime** - Proper enum object behavior instead of string conversion

### Technical Implementation

**Build System Updates:**
- Updated `pyproject.toml` with Pydantic `>=2.7,<3` constraint
- Added `plugins = ["pydantic.mypy"]` integration
- Enabled strict MyPy configuration with `disallow_any_generics=true`
- Added `py.typed` file for PEP 561 compliance

**Configuration Model Improvements:**
- Removed `use_enum_values=True` from all ConfigDict instances
- Added discriminated unions for shape-specific configurations:
  ```python
  ShapeSpecificConfig = Annotated[
      Union[BasicShapeConfig, RoundedShapeConfig, ConnectedShapeConfig],
      Field(discriminator="shape"),
  ]
  ```
- Updated enum comparisons to use `.value` only when string comparison needed
- Modern factory methods with proper type safety

**Type Safety Enhancements:**
- Created comprehensive TypedDict definitions in `segnomms/types.py`
- Updated shape renderers with type-safe patterns:
  ```python
  def render(
      self, x: float, y: float, size: float, **kwargs: Unpack[SquareRenderKwargs]
  ) -> ET.Element:
  ```
- Added proper type annotations throughout config modules

### Validation Status

**MyPy Compliance:**
- ✅ **segnomms/config/**: 9/9 files pass strict MyPy validation (100% clean)
- 🔄 **Overall codebase**: 215 MyPy errors remain (primarily in other modules, mostly annotation issues)

**Testing Status:**
- ✅ **Core functionality**: Configuration models work correctly with enum objects
- 🔄 **Test expectations**: Some tests still expect string values, need updating to expect enum objects
- ✅ **Backward compatibility**: Factory methods maintain compatibility with string inputs

### Best Practices Established

**For Future Development:**
1. **Always use enum objects** - Compare against `MergeStrategy.SOFT`, not `"soft"`
2. **Use discriminated unions** - For shape-specific configuration variants
3. **Apply TypedDict patterns** - For type-safe **kwargs usage
4. **Maintain strict MyPy** - All config modules must pass strict validation
5. **Test enum expectations** - Update test assertions to expect enum objects

### Achievement Impact

This modernization provides:
- **Enhanced Type Safety** - Catch more errors at development time
- **Better IDE Support** - Improved autocompletion and error detection
- **Runtime Enum Objects** - More robust enum handling without string conversion
- **Modern Pydantic Patterns** - Following current best practices
- **Discriminated Union Benefits** - Proper type narrowing for shape configurations

**The configuration system is now a model implementation of Pydantic v2 + MyPy best practices.**

---

## 🔧 Development Standards and Practices

**⚠️ IMPORTANT:** These development practices must be followed without exception.

### Git Commit Standards

**NEVER use `--no-verify` flag with git commits.**
- Pre-commit hooks exist to maintain code quality and consistency
- All formatting, linting, and type checking issues must be resolved before committing
- If pre-commit hooks fail, fix the underlying issues rather than bypassing them
- This ensures consistent code quality across all contributions

**Before committing:**
1. Run `black`, `isort`, and `flake8` to fix formatting issues
2. Ensure MyPy passes without errors
3. Fix any trailing whitespace or other formatting issues
4. Only commit when all pre-commit hooks pass successfully

### Code Quality Requirements

- **MyPy:** All code must pass strict type checking with zero errors
- **Black:** All Python code must be formatted with Black
- **isort:** All imports must be properly sorted and organized
- **flake8:** All code must pass linting checks
- **Tests:** All existing tests must pass before committing

---

## 📚 Documentation Strategy and Policy

**⚠️ IMPORTANT:** This project uses a **Sphinx-first documentation strategy** to prevent documentation sprawl.

### Primary Documentation: Sphinx/RST (docs/source/)

**All user-facing documentation MUST go in Sphinx:**
- API reference (auto-generated from docstrings)
- Configuration guides
- Usage examples
- Installation instructions
- Feature documentation

**Update Sphinx docs when:**
- Adding new features or configuration options
- Changing existing APIs
- Adding examples or tutorials
- Updating installation/setup procedures

### Allowed Markdown Files (Repository Only)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/systmms) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
