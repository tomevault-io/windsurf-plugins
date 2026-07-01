---
trigger: always_on
description: This file provides guidance to Claude Code when working with this Ansible learning repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this Ansible learning repository.

## Repository Overview

**30 Days of Ansible Bootcamp** is a comprehensive, community-focused educational project designed to teach Ansible from fundamentals to advanced concepts through a structured 30-day curriculum. This repository serves as the code companion to a YouTube video series and blog content.

**Project Purpose**: Enable learners to progress from zero Ansible knowledge to production-ready automation skills through hands-on examples, progressive lessons, and real-world use cases.

**Audience**: Beginners to intermediate DevOps engineers, system administrators, and automation enthusiasts learning Ansible.

**Format**: Multi-modal learning experience:
- **Video lessons**: YouTube playlist (techbeatly channel)
- **Written documentation**: README files with detailed explanations
- **Hands-on code**: Working playbooks, configurations, and examples
- **Practice exercises**: Real-world use cases

## Repository Structure

### Daily Lesson Organization (Day-01 through Day-30)

The repository contains 30 progressive lessons, each in its own directory:

```
Day-XX-Topic-Name/
├── README.md           # Lesson content, explanations, concepts
├── site.yaml          # Main example playbook (if applicable)
├── ansible.cfg        # Local Ansible configuration
├── inventory          # Sample inventory file
└── [additional files] # Supporting files (vars, templates, etc.)
```

**Learning Progression**:
- **Days 1-6**: Environment setup, installation, lab configuration
- **Days 7-10**: Fundamentals (inventory, ad-hoc commands, playbooks, privilege management)
- **Days 11-21**: Core concepts (modules, variables, facts, magic variables, vault/secrets)
- **Days 22-27**: Advanced control (loops, conditionals, handlers, failures, blocks, Jinja2)
- **Days 28-30**: Advanced topics (roles, parallelism, host patterns)

### Use Case Directories

Practical, real-world scenarios demonstrating specific Ansible applications:

- `Use-Case-Ansible-Variables/` - Variable management patterns
- `Use-Case-Calling-Role-with-Variable/` - Role parameterization
- `Use-Case-Collect-Host-Info/` - Fact gathering and reporting
- `Use-Case-Modify-JSON-YAML/` - Data manipulation
- `Use-Case-Vault-Advanced/` - Advanced secrets management with vault

Use cases include complete working examples with:
- Full playbook implementations
- Configuration files
- Sample data/variables
- vault-related scripts (for vault use cases)

### Work-in-Progress Content

`z_In_Prog/` - Contains draft lessons and experimental content:
- Lessons 7-12 (alternate numbering/organization)
- Content being developed or revised
- May not align with current day numbering

**Important**: This directory contains work that may be incomplete or superseded by content in the main Day-XX directories.

## Working with This Repository

### Running Examples

Each day's directory is **self-contained** with its own configuration:

```bash
# Navigate to specific day
cd Day-XX-Topic-Name/

# Run the example playbook
ansible-playbook site.yaml

# Or with explicit config
ansible-playbook -i inventory site.yaml
```

**Common Configuration Pattern**:
Most `ansible.cfg` files use:
```ini
[defaults]
inventory = ./inventory
remote_user = devops
host_key_checking = False
```

### Typical Inventory Format

Examples use simple INI-style inventories with test nodes:
```ini
node1.techbeatly.com
node2.techbeatly.com

[webservers]
node1.techbeatly.com

[databases]
node2.techbeatly.com
```

**Note**: These are example hostnames. Learners adapt these to their lab environment (VirtualBox/Vagrant VMs, cloud instances, etc.).

### Lab Environment Expectations

The curriculum assumes learners have set up a practice environment:
- **Control node**: Where Ansible is installed
- **Managed nodes**: 1-3 test systems (typically Linux)
- **Common setups**: VirtualBox + Vagrant (covered in Days 2-4), cloud VMs, containers

Most examples target:
- RHEL/CentOS-based systems (uses `yum` module)
- SSH-accessible nodes
- User `devops` with sudo privileges

## Content Development Guidelines

### When Adding or Modifying Lessons

**Maintain Progressive Learning**:
- Each day should build on previous concepts
- Avoid introducing advanced topics before prerequisites are covered
- Reference previous days when using earlier concepts

**Consistency Standards**:
1. **README Structure**: Start with YouTube video link, explain concept, show examples, provide full working code
2. **Playbook Style**:
   - Use descriptive task names
   - Include comments for complex logic
   - Follow YAML best practices (proper indentation, no tabs)
   - Start with `---`, ending `...` is optional
3. **File Naming**: Use `site.yaml` for main playbook, descriptive names for specialized playbooks
4. **Variables**: Use clear, descriptive variable names; document in README

**Code Quality for Teaching**:
- **Clarity over cleverness**: Choose readable code over concise-but-obscure patterns
- **Comment non-obvious behavior**: Explain *why*, not *what*
- **Show progression**: When showing multiple approaches, start simple then add complexity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iamgini/30-Days-of-Ansible-Bootcamp](https://github.com/iamgini/30-Days-of-Ansible-Bootcamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
