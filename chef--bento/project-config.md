---
trigger: always_on
description: This repository contains the Bento project, which provides:
---

# Bento Repository - Agent Instructions

This repository contains the Bento project, which provides:
1. **Ruby CLI Wrapper**: A Ruby gem (`bento`) that wraps Hashicorp Packer for building Vagrant boxes
2. **Packer Templates**: HCL-based Packer templates for building Vagrant base boxes across multiple operating systems and providers

## Repository Overview

### Ruby Component (CLI Wrapper)
- **Language**: Ruby 3.0+
- **Main Executable**: `bin/bento`
- **Library Code**: `lib/bento/`
- **Key Features**: Build, test, upload, list, and normalize Vagrant boxes
- **Dependencies**: mixlib-shellout, Packer, Vagrant

### Packer Component (Box Definitions)
- **Language**: HCL (HashiCorp Configuration Language)
- **Templates Directory**: `packer_templates/`
- **OS Variable Files**: `os_pkrvars/` (organized by OS family)
- **Supported Providers**: VirtualBox, VMware, Parallels, UTM, QEMU, Hyper-V
- **Supported Architectures**: x86_64, aarch64

## Code Change Requirements

### For ALL Changes to This Repository:

#### 1. Linting Requirements
Before committing any changes, you **MUST** run the following linters based on the files modified:

**Ruby Files** (`*.rb`, `Rakefile`, `*.gemspec`):
```bash
bundle install
bundle exec cookstyle -a .
```

**Markdown Files** (`*.md`):
```bash
# Markdown linting is performed via GitHub Actions
# Local testing not required but verify links work
```

**YAML Files** (`*.yml`, `*.yaml`):
```bash
# YAML linting is performed via GitHub Actions
# Ensure valid YAML syntax
```

**Shell Scripts** (`*.sh`):
```bash
shellcheck *.sh
```

**PowerShell Scripts** (`*.ps1`):
```bash
# PowerShell linting is performed via GitHub Actions
```

**Packer Templates** (`*.pkr.hcl`, `*.pkrvars.hcl`):
```bash
cd /path/to/bento
packer init -upgrade ./packer_templates
packer validate -var-file=os_pkrvars/<os>/<template>.pkrvars.hcl ./packer_templates
```

**All Packer Templates (Comprehensive Validation)**:
```bash
bundle exec rake validate
```

#### 2. Testing Requirements

**For Ruby Code Changes**:
```bash
# Install all dependencies (includes test-kitchen, kitchen-vagrant, and rspec)
bundle install

# Run the RSpec unit test suite
bundle exec rspec
# or equivalently:
bundle exec rake spec
# (bundle exec rake also runs spec by default)

# Run the bento CLI commands to verify functionality
bento list
bento build --dry-run os_pkrvars/<test-template>.pkrvars.hcl
```

**RSpec test suite** is located in `spec/` and covers all library modules:
- `spec/bento/common_spec.rb` — Common module helpers
- `spec/bento/buildmetadata_spec.rb` — BuildMetadata
- `spec/bento/providermetadata_spec.rb` — ProviderMetadata
- `spec/bento/packerexec_spec.rb` — PackerExec
- `spec/bento/runner_spec.rb` — BuildRunner
- `spec/bento/test_spec.rb` — TestRunner
- `spec/bento/upload_spec.rb` — UploadRunner

**For Packer Template Changes**:
```bash
# Validate the specific template
packer validate -var-file=os_pkrvars/<os>/<modified-template>.pkrvars.hcl ./packer_templates

# If possible, run a test build (requires hypervisor)
bento build os_pkrvars/<os>/<modified-template>.pkrvars.hcl

# Test with kitchen (if applicable)
bento test
```

#### 3. Documentation Updates

**README.md** - Update if:
- Adding new features to the bento CLI
- Changing CLI command syntax or options
- Adding/removing supported operating systems
- Modifying build requirements or prerequisites
- Changing installation instructions

**CHANGELOG.md** - Update for:
- **ALL user-facing changes** (bug fixes, features, breaking changes)
- New OS support or version updates
- Provider support changes (VirtualBox, VMware, etc.)
- Ruby gem version changes
- Template modifications affecting builds

**Format for CHANGELOG.md**:
```markdown
## Builds for version <VERSION>

[Add/update the build matrix table showing supported OS and providers]

### Changes
- Added support for [OS Name] version [X.Y]
- Fixed [issue description]
- Updated [component] to improve [functionality]
- BREAKING: [description of breaking change]
```

**TESTING.md** - Update if:
- Changing test procedures
- Adding new test requirements
- Modifying how `bento test` works

## Key Files to Understand

### Ruby Codebase
- `lib/bento/cli.rb` - Command-line interface definitions
- `lib/bento/runner.rb` - Build orchestration logic
- `lib/bento/packerexec.rb` - Packer command execution
- `lib/bento/test.rb` - Test kitchen integration (uses Kitchen Ruby API directly via `require 'kitchen'`; no shell-out)
- `lib/bento/upload.rb` - Vagrant Cloud upload logic
- `lib/bento/buildmetadata.rb` - Metadata generation
- `bento.gemspec` - Gem specification and dependencies (`test-kitchen` and `kitchen-vagrant` are runtime deps)
- `spec/` - RSpec unit tests for all library modules
- `.rspec` - RSpec configuration (documentation format, color output)

### Packer Templates
- `packer_templates/pkr-variables.pkr.hcl` - Variable definitions
- `packer_templates/pkr-sources.pkr.hcl` - Provider source configurations
- `packer_templates/pkr-builder.pkr.hcl` - Build orchestration
- `packer_templates/pkr-plugins.pkr.hcl` - Required Packer plugins
- `os_pkrvars/<os>/<os>-<version>-<arch>.pkrvars.hcl` - OS-specific variables

### Configuration Files
- `builds.yml` - Build configuration (public boxes, architectures, exclusions)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chef/bento](https://github.com/chef/bento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
