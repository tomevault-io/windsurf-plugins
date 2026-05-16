---
trigger: always_on
description: This document captures best practices and preferences for working on this project, particularly for Ansible infrastructure automation.
---

# Claude Code Guide for gossip_observer

This document captures best practices and preferences for working on this project, particularly for Ansible infrastructure automation.

## Ansible Playbook Best Practices

### Template vs. Copy Module

**Always use `ansible.builtin.template` for .j2 files**, even if they don't contain variables:

```yaml
# ✅ Correct
- name: Install systemd template
  ansible.builtin.template:
    src: gossip-collector@.service.j2
    dest: /etc/systemd/system/gossip-collector@.service

# ❌ Wrong
- name: Install systemd template
  ansible.builtin.copy:
    src: ../templates/gossip-collector@.service.j2
    dest: /etc/systemd/system/gossip-collector@.service
```

**Use just the filename** - Ansible automatically looks in the `templates/` directory. Never use relative paths like `../templates/`.

### Idempotency

**Always make tasks as idempotent as possible:**

1. **Conditional daemon reloads** - Only reload systemd when unit files change:

```yaml
- name: Install systemd template
  ansible.builtin.template:
    src: service.j2
    dest: /etc/systemd/system/service.service
  register: systemd_template_changed

- name: Reload systemd daemon
  ansible.builtin.systemd:
    daemon_reload: true
  when: systemd_template_changed is changed
```

1. **Config/binary changes trigger restarts** - Always add `notify:` to ensure services restart when their config or binary changes:

```yaml
- name: Deploy config file
  ansible.builtin.template:
    src: config.toml.j2
    dest: /etc/service/config.toml
  notify: Restart service

- name: Deploy binary
  ansible.builtin.copy:
    src: "{{ binary_path }}"
    dest: /usr/local/bin/service
  notify: Restart service
```

### Handlers

**Handler names cannot use loop variables:**

```yaml
# ❌ Wrong - handler names can't be templated
- name: Restart collector-{{ item.uuid }}
  ansible.builtin.systemd:
    name: "collector-{{ item.uuid }}"

# ✅ Correct - handler loops internally
- name: Restart all collectors
  ansible.builtin.systemd:
    name: "collector-{{ item.uuid }}"
    state: restarted
  loop: "{{ collectors }}"
```

Tasks notify handlers with static names; handlers define their own loops.

### UFW Firewall Rules

**Rule order matters** - ALLOW rules must come before DENY rules:

1. **Task execution order sets initial rule order** - UFW appends rules in the order Ansible adds them
2. **Use comments** to identify rules: `comment: "Service-purpose"`
3. **The `insert` parameter** only works when adding NEW rules; existing rules won't be repositioned

```yaml
# ALLOW rules first
- name: Allow SSH
  community.general.ufw:
    rule: limit
    port: 22
    comment: "SSH"

- name: Allow service ports
  community.general.ufw:
    rule: allow
    port: "{{ item.port }}"
    comment: "Service-{{ item.name }}"
  loop: "{{ services }}"

# DENY rule last
- name: Deny all other traffic
  community.general.ufw:
    rule: deny
    comment: "Deny-public"
```

Verify with `sudo ufw status numbered` after deployment.

### Loop Patterns

**Cartesian product for nested iteration:**

```yaml
# Create multiple directories for multiple instances
- name: Create instance directories
  ansible.builtin.file:
    path: "{{ item[0] }}/{{ item[1].uuid }}"
    state: directory
  loop: "{{ ['/var/lib/service', '/etc/service'] | product(instances) }}"
```

The `product()` filter creates all combinations: `[path1, instance1], [path1, instance2], [path2, instance1], [path2, instance2]`

### File Content Formatting

**Text files need trailing newlines:**

```yaml
# ✅ Add \n for proper Unix text file format
- name: Deploy mnemonic
  ansible.builtin.copy:
    content: "{{ mnemonic }}\n"
    dest: /path/to/mnemonic.txt
```

This works correctly with Rust's `.trim()` which strips the newline before parsing.

### TOML Template Values

**Quote string values in Jinja2 TOML templates:**

TOML has distinct types - strings must be quoted, but integers and booleans must not:

```jinja2
# Strings (IPs, URLs, paths) - MUST quote
tor_proxy_addr = "{{ item.tor_proxy_addr | default('127.0.0.1') }}"
server_url = "{{ server_url }}"

# Integers - NO quotes
listen_port = {{ item.port | default(9735) }}

# Booleans - NO quotes
enable_tor = {{ item.enable_tor | default(true) }}
```

Unquoted `127.0.0.1` causes TOML to parse it as an invalid float (multiple decimal points).

### Systemd Service Templates

**Use systemd template units for multi-instance services:**

- Template file: `service@.service`
- Instance specifier: `%i` (becomes the UUID or instance name)
- Enable instances: `systemctl enable service@{uuid}.service`

**Add randomized startup delays** to prevent thundering herd:

```ini
[Service]
RandomizedDelaySec=120  # For collectors with many instances
RandomizedDelaySec=30   # For controllers with few instances
```

Spreads startups over 0-N seconds to avoid simultaneous resource access. Use larger values (120s) for services with many instances that may restart together (collectors). Use smaller values (30s) for services with few instances (controllers, archivers) where faster startup is preferred.

## Rust Code Patterns

### sqlx Query Macros vs Runtime Queries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jharveyb/gossip_observer](https://github.com/jharveyb/gossip_observer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
