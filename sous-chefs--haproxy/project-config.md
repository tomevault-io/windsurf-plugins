---
trigger: always_on
description: **ALWAYS follow these instructions first and fallback to additional search and context gathering only if the information here is incomplete or found to be in error.**
---

# HAProxy Cookbook Development Instructions

**ALWAYS follow these instructions first and fallback to additional search and context gathering only if the information here is incomplete or found to be in error.**

This is a Chef cookbook for installing and configuring HAProxy load balancer. It provides custom Chef resources for managing HAProxy installations, configurations, and services across multiple platforms.

## Working Effectively

### Bootstrap Environment and Dependencies
```bash
# Install Chef development tools (choose one method):
# Method 1: Install Chef Workstation (recommended)
wget https://packages.chef.io/files/stable/chef-workstation/24.12.1031/ubuntu/24.04/chef-workstation_24.12.1031-1_amd64.deb
sudo dpkg -i chef-workstation_24.12.1031-1_amd64.deb

# Method 2: Install individual gems if Chef Workstation fails
sudo gem install cookstyle chefspec rspec test-kitchen kitchen-dokken --no-document
```

### Core Development Commands
```bash
# Install cookbook dependencies
chef install Policyfile.rb

# Lint Ruby code - takes 2-3 seconds - NEVER CANCEL
cookstyle .

# Fix auto-correctable linting issues
cookstyle -a .

# Run unit tests - takes 30 seconds to 2 minutes - NEVER CANCEL
# NOTE: Requires full Chef installation with chefspec gem
rspec spec/

# Run all unit tests with detailed output
rspec spec/ --format documentation
```

### Integration Testing (Advanced)
**CRITICAL**: Integration tests require Docker and take 15-45 minutes per suite. NEVER CANCEL.
**NOTE**: Requires test-kitchen and kitchen-dokken gems installed.

```bash
# List all available test suites (requires test-kitchen installation)
kitchen list

# Test a specific suite - NEVER CANCEL: Takes 15-45 minutes
# Set timeout to 60+ minutes for any kitchen commands
kitchen test package-ubuntu-2204

# Test source compilation (takes longest) - NEVER CANCEL: Takes 30-60 minutes  
kitchen test source-default-ubuntu-2204

# Destroy test instances after testing
kitchen destroy
```

### Build Validation Workflow
**Minimal validation** (works in any environment with cookstyle):
```bash
# 1. Lint code (~2-3 seconds) - ALWAYS WORKS
cookstyle .

# 2. Fix linting issues automatically - ALWAYS WORKS  
cookstyle -a .

# 3. Verify Ruby syntax - ALWAYS WORKS
ruby -c resources/install.rb
ruby -c resources/service.rb
```

**Full validation** (requires complete Chef environment):
```bash
# 1. Lint code (~2-3 seconds)
cookstyle .

# 2. Run unit tests (~30 seconds - 2 minutes) - NEVER CANCEL
rspec spec/

# 3. Test basic package installation (15-30 minutes) - NEVER CANCEL
kitchen test package-ubuntu-2204

# 4. Clean up test instances
kitchen destroy
```

## Timing Expectations and Timeouts
- **Linting (`cookstyle`)**: 2-3 seconds
- **Unit tests (`rspec`)**: 30 seconds to 2 minutes - NEVER CANCEL
- **Single integration test**: 15-45 minutes - NEVER CANCEL - Set timeout to 60+ minutes
- **Source compilation tests**: 30-60 minutes - NEVER CANCEL - Set timeout to 90+ minutes  
- **Full CI matrix**: 1-2 hours across all platforms and suites

**CRITICAL**: Always set timeouts of 60+ minutes for kitchen commands and 30+ minutes for unit tests.

## Manual Validation Scenarios

After making changes, ALWAYS test at least one complete scenario:

### Package Installation Validation
```bash
# Test the most common installation method
kitchen converge package-ubuntu-2204
kitchen verify package-ubuntu-2204
kitchen destroy package-ubuntu-2204
```

### Source Compilation Validation  
```bash
# Test source compilation (most complex scenario)
kitchen converge source-default-ubuntu-2204
kitchen verify source-default-ubuntu-2204
kitchen destroy source-default-ubuntu-2204
```

### Configuration Validation
```bash
# Test configuration management
kitchen converge config-2-ubuntu-2204
kitchen verify config-2-ubuntu-2204
kitchen destroy config-2-ubuntu-2204
```

## Repository Structure and Navigation

### Key Directories
- `resources/` - Custom Chef resources (install.rb, service.rb, config_global.rb, etc.)
- `libraries/` - Helper modules and shared code
- `test/cookbooks/test/recipes/` - Test recipes for integration testing
- `test/integration/` - InSpec integration test controls
- `spec/unit/` - ChefSpec unit tests
- `templates/` - ERB templates for configuration files
- `documentation/` - Resource documentation

### Important Files
- `metadata.rb` - Cookbook metadata and dependencies
- `Policyfile.rb` - Cookbook dependency management
- `kitchen.yml` - Vagrant-based integration testing (local development)
- `kitchen.dokken.yml` - Docker-based integration testing (CI)
- `.rubocop.yml` - Ruby linting configuration
- `.github/workflows/ci.yml` - Continuous integration pipeline

### Common Test Suites
- `package` - Test package installation method
- `source-default` - Test default source compilation
- `source-24`, `source-26`, `source-28` - Test specific HAProxy versions
- `config-2`, `config-acl`, `config-ssl-redirect` - Test various configurations
- `source-lua` - Test Lua compilation support

## Cookbook Architecture

### Custom Resources Available
- `haproxy_install` - Install HAProxy via package or source
- `haproxy_service` - Manage HAProxy service

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sous-chefs/haproxy](https://github.com/sous-chefs/haproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
