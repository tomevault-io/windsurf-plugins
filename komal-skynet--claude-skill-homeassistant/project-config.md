---
trigger: always_on
description: Expert-level Home Assistant configuration management with efficient deployment workflows (git and rapid scp iteration), remote CLI access via SSH and hass-cli, automation verification protocols, log analysis, reload vs restart optimization, and comprehensive Lovelace dashboard management for tablet-optimized UIs. Includes template patterns, card types, debugging strategies, and real-world examples.
---


# Home Assistant Manager

Expert-level Home Assistant configuration management with efficient workflows, remote CLI access, and verification protocols.

## Core Capabilities

- Remote Home Assistant instance management via SSH and hass-cli
- Smart deployment workflows (git-based and rapid iteration)
- Configuration validation and safety checks
- Automation testing and verification
- Log analysis and error detection
- Reload vs restart optimization
- Lovelace dashboard development and optimization
- Template syntax patterns and debugging
- Tablet-optimized UI design

## Prerequisites

Before starting, verify the environment has:
1. SSH access to Home Assistant instance (`root@homeassistant.local`)
2. `hass-cli` installed locally
3. Environment variables loaded (HASS_SERVER, HASS_TOKEN)
4. Git repository connected to HA `/config` directory
5. Context7 MCP server with Home Assistant docs (recommended)

## Remote Access Patterns

### Using hass-cli (Local, via REST API)

All `hass-cli` commands use environment variables automatically:

```bash
# List entities
hass-cli state list

# Get specific state
hass-cli state get sensor.entity_name

# Call services
hass-cli service call automation.reload
hass-cli service call automation.trigger --arguments entity_id=automation.name
```

### Using SSH for HA CLI

```bash
# Check configuration validity
ssh root@homeassistant.local "ha core check"

# Restart Home Assistant
ssh root@homeassistant.local "ha core restart"

# View logs
ssh root@homeassistant.local "ha core logs"

# Tail logs with grep
ssh root@homeassistant.local "ha core logs | grep -i error | tail -20"
```

## Deployment Workflows

### Standard Git Workflow (Final Changes)

Use for changes you want in version control:

```bash
# 1. Make changes locally
# 2. Check validity
ssh root@homeassistant.local "ha core check"

# 3. Commit and push
git add file.yaml
git commit -m "Description"
git push

# 4. CRITICAL: Pull to HA instance
ssh root@homeassistant.local "cd /config && git pull"

# 5. Reload or restart
hass-cli service call automation.reload  # if reload sufficient
# OR
ssh root@homeassistant.local "ha core restart"  # if restart needed

# 6. Verify
hass-cli state get sensor.new_entity
ssh root@homeassistant.local "ha core logs | grep -i error | tail -20"
```

### Rapid Development Workflow (Testing/Iteration)

Use `scp` for quick testing before committing:

```bash
# 1. Make changes locally
# 2. Quick deploy
scp automations.yaml root@homeassistant.local:/config/

# 3. Reload/restart
hass-cli service call automation.reload

# 4. Test and iterate (repeat 1-3 as needed)

# 5. Once finalized, commit to git
git add automations.yaml
git commit -m "Final tested changes"
git push
```

**When to use scp:**
- 🚀 Rapid iteration and testing
- 🔄 Frequent small adjustments
- 🧪 Experimental changes
- 🎨 UI/Dashboard work

**When to use git:**
- ✅ Final tested changes
- 📦 Version control tracking
- 🔒 Important configs
- 👥 Changes to document

## Reload vs Restart Decision Making

**ALWAYS assess if reload is sufficient before requiring a full restart.**

### Can be reloaded (fast, preferred):
- ✅ Automations: `hass-cli service call automation.reload`
- ✅ Scripts: `hass-cli service call script.reload`
- ✅ Scenes: `hass-cli service call scene.reload`
- ✅ Template entities: `hass-cli service call template.reload`
- ✅ Groups: `hass-cli service call group.reload`
- ✅ Themes: `hass-cli service call frontend.reload_themes`

### Require full restart:
- ❌ Min/Max sensors and platform-based sensors
- ❌ New integrations in configuration.yaml
- ❌ Core configuration changes
- ❌ MQTT sensor/binary_sensor platforms

## Automation Verification Workflow

**ALWAYS verify automations after deployment:**

### Step 1: Deploy
```bash
git add automations.yaml && git commit -m "..." && git push
ssh root@homeassistant.local "cd /config && git pull"
```

### Step 2: Check Configuration
```bash
ssh root@homeassistant.local "ha core check"
```

### Step 3: Reload
```bash
hass-cli service call automation.reload
```

### Step 4: Manually Trigger
```bash
hass-cli service call automation.trigger --arguments entity_id=automation.name
```

**Why trigger manually?**
- Instant feedback (don't wait for scheduled triggers)
- Verify logic before production
- Catch errors immediately

### Step 5: Check Logs
```bash
sleep 3
ssh root@homeassistant.local "ha core logs | grep -i 'automation_name' | tail -20"
```

**Success indicators:**
- `Initialized trigger AutomationName`
- `Running automation actions`
- `Executing step ...`
- No ERROR or WARNING messages

**Error indicators:**
- `Error executing script`
- `Invalid data for call_service`
- `TypeError`, `Template variable warning`

### Step 6: Verify Outcome

**For notifications:**
- Ask user if they received it
- Check logs for mobile_app messages

**For device control:**
```bash
hass-cli state get switch.device_name
```

**For sensors:**
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [komal-SkyNET/claude-skill-homeassistant](https://github.com/komal-SkyNET/claude-skill-homeassistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
