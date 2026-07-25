---
trigger: always_on
description: Azure Monitor Baseline Alerts (AMBA) is a comprehensive repository providing baseline alerting guidance and Infrastructure as Code (IaC) templates for Azure resources. The project includes a Hugo-based documentation website, Azure ARM/Bicep templates, and Python tooling for alert management.
---

# Azure Monitor Baseline Alerts (AMBA)

Azure Monitor Baseline Alerts (AMBA) is a comprehensive repository providing baseline alerting guidance and Infrastructure as Code (IaC) templates for Azure resources. The project includes a Hugo-based documentation website, Azure ARM/Bicep templates, and Python tooling for alert management.

**CRITICAL**: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap, Build, and Test the Repository

**NEVER CANCEL any build or test commands. All timing estimates include safety margins.**

1. **Install Hugo Extended (Required for documentation site)**:
   ```bash
   wget -O /tmp/hugo.deb https://github.com/gohugoio/hugo/releases/download/v0.117.0/hugo_extended_0.117.0_linux-amd64.deb
   sudo dpkg -i /tmp/hugo.deb
   hugo version  # Verify installation
   ```

2. **Install Python dependencies**:
   ```bash
   # For alert export tooling
   cd tooling/export-alerts
   pip install -r requirements.txt  # Installs pyyaml, openpyxl

   # For template generation
   cd ../generate-templates
   pip install -r requirements.txt  # Installs pyyaml
   ```

3. **Build Hugo documentation site**:
   ```bash
   cd /path/to/repo/root
   hugo --gc --minify
   ```
   - **Duration**: 15 seconds - NEVER CANCEL. Set timeout to 60+ seconds.
   - **Output**: Static site files in `./public/` directory
   - **Pages generated**: ~432 pages, ~735 non-page files, ~3001 static files

4. **Export alert definitions** (validates Python tooling):
   ```bash
   cd tooling/export-alerts
   python export-alerts.py --path ../../services --template ./alerts-template.xlsx --output-xls ../../services/amba-alerts.xlsx --output-json ../../services/amba-alerts.json --output-yaml ../../services/amba-alerts.yaml
   ```
   - **Duration**: 5 seconds - NEVER CANCEL. Set timeout to 30+ seconds.
   - **Output**: Generates Excel, JSON, and YAML files with consolidated alert definitions

### Run the Development Environment

**ALWAYS run the bootstrapping steps first.**

1. **Run Hugo development server**:
   ```bash
   hugo server -D
   ```
   - **Duration**: 15 seconds initial build - NEVER CANCEL. Set timeout to 60+ seconds.
   - **Access**: http://localhost:1313/azure-monitor-baseline-alerts/
   - **Note**: Server watches for changes and rebuilds automatically

2. **Test Bicep compilation**:
   ```bash
   # Example: Compile policy set definitions
   bicep build patterns/alz/templates/policySets.bicep --outfile /tmp/policySets.json

   # Example: Compile network policies
   bicep build patterns/alz/templates/policies-Network.bicep --outfile /tmp/policies-Network.json
   ```
   - **Duration**: 3-5 seconds per file - NEVER CANCEL. Set timeout to 30+ seconds.

## Validation

**ALWAYS manually validate any new code via the following steps after making changes.**

### Required Validation Steps

1. **YAML validation** (for alert definitions):
   ```bash
   python .github/yml-schemas/validate_yml.py "path/to/alerts.yaml"
   ```
   - **Duration**: <5 seconds - Set timeout to 30+ seconds.
   - **Purpose**: Validates alert definition schema compliance

2. **Hugo site build validation**:
   ```bash
   hugo --gc --minify
   ```
   - **Duration**: 15 seconds - NEVER CANCEL. Set timeout to 60+ seconds.
   - **Purpose**: Ensures documentation builds successfully

3. **Alert export validation**:
   ```bash
   cd tooling/export-alerts
   python export-alerts.py --path ../../services --template ./alerts-template.xlsx --output-xls /tmp/test-alerts.xlsx --output-json /tmp/test-alerts.json --output-yaml /tmp/test-alerts.yaml
   ```
   - **Duration**: 5 seconds - Set timeout to 30+ seconds.
   - **Purpose**: Ensures alert processing tooling works correctly

### Manual Testing Scenarios

**ALWAYS test these scenarios after making changes to ensure functionality:**

1. **Documentation Site Navigation**:
   - Start Hugo server: `hugo server -D`
   - Navigate to http://localhost:1313/azure-monitor-baseline-alerts/
   - Test key pages: Welcome, Patterns/ALZ, Services sections
   - Verify search functionality works

2. **Alert Definition Processing**:
   - Modify an alert in `services/{service}/{resource}/alerts.yaml`
   - Run export script to verify processing works
   - Check generated outputs contain your changes

3. **Bicep Template Validation**:
   - Make changes to templates in `patterns/alz/templates/`
   - Compile with `bicep build` to verify syntax
   - Check for any compilation errors

## CI Validation Requirements

**Always run these commands before committing to ensure CI will pass:**

1. **YAML Schema Validation**:
   ```bash
   # Test changed YAML files
   python .github/yml-schemas/validate_yml.py "path/to/changed/alerts.yaml"
   ```

2. **Hugo Build Check**:
   ```bash
   hugo --gc --minify
   ```
   - **Must complete successfully** - site must build without errors

3. **Bicep Compilation Check**:
   ```bash
   # For any modified .bicep files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/azure-monitor-baseline-alerts](https://github.com/Azure/azure-monitor-baseline-alerts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
