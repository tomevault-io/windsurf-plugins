---
trigger: always_on
description: > **🚨 THIS FILE IS FOR FRAMEWORK DEVELOPERS ONLY**
---

# Claude Multi-Agent PM Framework Development Rules v1.2.5

> **🚨 THIS FILE IS FOR FRAMEWORK DEVELOPERS ONLY**
> 
> **This file contains development rules for contributors working on the claude-multiagent-pm framework codebase itself.**
> 
> **📍 If you are USING the framework:**
> - DO NOT follow these rules - they are for framework development only
> - Look for `framework/CLAUDE.md` in your deployed project for usage instructions
> - These rules apply ONLY to the framework source code repository
> 
> **📍 If you are DEVELOPING the framework:**
> - Follow ALL rules below strictly
> - Maintain clean root directory structure
> - Run comprehensive tests before commits
> - Preserve backward compatibility

---

## 📂 ROOT DIRECTORY HYGIENE RULES

### ✅ ALLOWED ROOT DOCUMENTS (ONLY THESE)
1. **CLAUDE.md** - Framework development rules (this file)
2. **README.md** - Framework overview and quick start
3. **CHANGELOG.md** - Version history and changes
4. **RELEASE_NOTES.md** - Detailed release information

### ⛔ STRICT DIRECTORY ORGANIZATION
- **ALL tests** → `tests/` directory (NO test files in root)
- **ALL documentation** → `docs/` directory (except the 4 allowed root files)
- **ALL scripts** → `scripts/` directory
- **ALL build artifacts** → `.gitignore` them (never commit)
- **NO temporary files** in root directory
- **NO example files** in root directory
- **NO generated reports** in root directory

### 🧹 ROOT CLEANUP CHECKLIST
Before committing, ensure:
- [ ] Only 4 allowed .md files exist in root
- [ ] All test files are in `tests/`
- [ ] All docs are in `docs/`
- [ ] No temporary or generated files in root
- [ ] `.gitignore` is properly configured

---

## 🔄 DEVELOPMENT WORKFLOW

### 1. Feature Development Process
```bash
# 1. Create feature branch
git checkout -b feature/your-feature-name

# 2. Make changes following all rules
# 3. Run comprehensive tests
pytest tests/
npm test

# 4. Verify root directory hygiene
ls -la | grep -E '\.(md|py|js|json)$'  # Should show only allowed files

# 5. Run integrity checks
python scripts/test_framework_integrity.py
python scripts/validate_version_consistency.py

# 6. Commit with conventional commits
git commit -m "feat: Add new capability" 
# or "fix:", "docs:", "test:", "refactor:", "chore:"

# 7. Push and create PR
git push origin feature/your-feature-name
```

### 2. 3-Stage Deployment Model

The framework follows a **3-stage deployment model** for all changes:

#### Stage 1: Development (Automatic)
- Changes in source code are immediately available
- Run tests and verify functionality
- Use: `./scripts/deploy-3stage.sh dev`

#### Stage 2: Local Machine Deployment
- Deploy changes to your local machine for testing
- Updates `~/.local/bin/claude-pm` and Python packages
- Use: `./scripts/deploy-3stage.sh local`

#### Stage 3: Publish
- Release to npm and PyPI registries
- Only after thorough local testing
- Use: `./scripts/deploy-3stage.sh publish`

**📚 Full deployment documentation**: See [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md#3-stage-deployment-model)

### 3. Testing Requirements
**MANDATORY before ANY commit:**
- ✅ Unit tests pass: `pytest tests/unit/`
- ✅ Integration tests pass: `pytest tests/integration/`
- ✅ E2E tests pass: `pytest tests/e2e/`
- ✅ Framework integrity validated
- ✅ Version consistency verified
- ✅ Root directory hygiene maintained

### 4. Code Review Checklist
- [ ] No files in root except 4 allowed .md files
- [ ] All tests in `tests/` directory
- [ ] All docs in `docs/` directory
- [ ] No deployment code mixed with development code
- [ ] Framework template (`framework/CLAUDE.md`) unchanged
- [ ] Version files synchronized if version changed
- [ ] Backward compatibility maintained

---

## 👥 CONTRIBUTION GUIDELINES

### Pull Request Requirements
1. **Title**: Use conventional commit format
2. **Description**: Explain what and why (not how)
3. **Tests**: Include tests for new features
4. **Documentation**: Update relevant docs
5. **Breaking Changes**: Clearly marked and justified

### Coding Standards
- **Python**: Follow PEP 8, use type hints
- **JavaScript**: Follow ESLint configuration
- **Markdown**: Use consistent formatting
- **File Organization**: Respect directory structure

### Documentation Standards
- **Root Docs**: Only 4 allowed files, keep concise
- **Feature Docs**: Detailed docs go in `docs/features/`
- **API Docs**: Technical docs go in `docs/technical/`
- **Examples**: All examples go in `docs/examples/`

---

## 🚨 CRITICAL FRAMEWORK PROTECTION RULES

### ⛔ ABSOLUTE PROHIBITIONS - NEVER DO THESE

1. **NEVER DELETE OR MODIFY `framework/CLAUDE.md`**
   - This is the master template for ALL framework deployments
   - Protected by automatic backup system (keeps 2 most recent copies)
   - Any changes must go through proper version control and testing
   - **CRITICAL**: This file is ESSENTIAL to framework operation and MUST NOT be deleted by cleanup processes
   - **WARNING**: Deletion of this file will break ALL framework deployments across projects

2. **NEVER REMOVE PROTECTION MECHANISMS**
   - `_protect_framework_template()` method must remain intact
   - `_backup_framework_template()` functionality is critical
   - Framework integrity validation must stay enabled

3. **NEVER BYPASS VERSION CHECKING**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bobmatnyc/claude-multiagent-pm](https://github.com/bobmatnyc/claude-multiagent-pm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
