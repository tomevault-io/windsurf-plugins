---
trigger: always_on
description: **Purpose:** This file provides AI assistant context for executing threat hunting queries against Splunk using ATHF's native API integration.
---

# AGENTS.md - Splunk Integration Context for AI Assistants

**Purpose:** This file provides AI assistant context for executing threat hunting queries against Splunk using ATHF's native API integration.

---

## Query Execution Method

**PRIMARY METHOD: Use `athf splunk search` CLI command**

AI assistants MUST use the `athf splunk search` command for all Splunk query execution. This provides:
- Automatic authentication via environment variables
- Result limits and timeouts
- Structured output (JSON, table, raw)
- Error handling and retry logic

**✅ CORRECT - Use CLI:**
```bash
athf splunk search 'index=windows EventCode=4624 | stats count by ComputerName' \
  --earliest "-7d" \
  --count 100 \
  --format json
```

**❌ WRONG - Don't use raw HTTP requests or manual API calls**

---

## Default Index

**⚠️ CRITICAL: For this deployment, always use `index=thrunt` for all hunt queries.**

The `thrunt` index contains 12M+ security events including:
- **Windows events** (XmlWinEventLog) - 513K events
- **Network traffic** (stream:ip, stream:dns, stream:tcp, stream:http)
- **Firewall logs** (juniper:junos:firewall) - 535K events
- **Linux audit logs** - 131K events
- **Cloud logs** (AWS, Azure, O365, Kubernetes)

**Always start queries with:** `index=thrunt`

---

## SPL Query Syntax Guidelines

### Time Bounds (Mandatory)

**ALWAYS specify time range** using `--earliest` and `--latest` flags:

```bash
# Last 7 days (default for exploratory hunts)
--earliest "-7d" --latest "now"

# Last 24 hours (for initial testing)
--earliest "-24h" --latest "now"

# Last hour (for quick tests)
--earliest "-1h" --latest "now"

# All available data in thrunt index (use sparingly)
--earliest "0" --latest "now"
```

**Note:** The `thrunt` index contains historical data. Use `--earliest "0"` to search all data, but be mindful of query performance.

### Result Limits (Mandatory)

**ALWAYS use `--count` flag** to limit results:

```bash
# Start with 100 for exploratory queries
--count 100

# Increase if needed based on COUNT-FIRST results
--count 1000

# For COUNT queries only
--count 0  # (only when using | stats count)
```

### COUNT-FIRST Strategy (Required)

**ALWAYS count before pulling results.** Execute queries in this sequence:

#### Step 1: Baseline Count
```bash
athf splunk search 'index=thrunt sourcetype="XmlWinEventLog" EventCode=4624 | stats count' \
  --earliest "0" \
  --count 1 \
  --format json
```

**Decision tree:**
- **count = 0** → No data, stop
- **count < 100** → Proceed with `--count 100`
- **count 100-1000** → Proceed with `--count 1000`
- **count > 1000** → Add filters, then recount

#### Step 2: Filtered Count (Add Constraints)
```bash
athf splunk search 'index=thrunt sourcetype="XmlWinEventLog" EventCode=4624 LogonType=10 | stats count' \
  --earliest "0" \
  --count 1 \
  --format json
```

#### Step 3: Pull Results (Only if Justified)
```bash
athf splunk search 'index=thrunt sourcetype="XmlWinEventLog" EventCode=4624 LogonType=10' \
  --earliest "0" \
  --count 100 \
  --format json
```

**STOP after each query** - Wait for user feedback before proceeding.

### Query Performance Best Practices

1. **Index + Sourcetype First** - Most efficient filters
   ```spl
   index=thrunt sourcetype="XmlWinEventLog" EventCode=4624
   ```

2. **Avoid Wildcards in Index** - Use specific index names
   ```spl
   ✅ index=thrunt
   ❌ index=*
   ```

3. **Use stats Instead of Head** - More efficient for large datasets
   ```spl
   ✅ | stats count by field
   ❌ | head 1000 | dedup field
   ```

4. **Field Extraction Efficiency** - Reference fields that exist in sourcetype
   ```spl
   ✅ EventCode=4624  (indexed field)
   ❌ custom_field=*  (may require extraction)
   ```

5. **Use Async for Long Queries** - If query may take >30 seconds
   ```bash
   athf splunk search 'complex query' \
     --async-search \
     --max-wait 600
   ```

---

## Common Field Names

### Windows Event Logs (sourcetype="XmlWinEventLog")

| Field | Description | Example Values |
|-------|-------------|----------------|
| `EventCode` | Windows Event ID | 4624, 4625, 4648, 4768 |
| `ComputerName` | Hostname | WIN-SERVER01 |
| `Account_Name` | Username | jdoe, administrator |
| `LogonType` | Windows logon type | 2 (interactive), 3 (network), 10 (RDP) |
| `SourceNetworkAddress` | Source IP address | 10.0.1.50 |
| `TargetUserName` | Target account | domain\user |
| `WorkstationName` | Source workstation | CLIENT01 |

**Common EventCodes:**
- **4624** - Successful logon
- **4625** - Failed logon
- **4648** - Logon using explicit credentials
- **4768** - Kerberos TGT requested
- **4769** - Kerberos service ticket requested
- **4771** - Kerberos pre-authentication failed

### Network Logs (sourcetype="stream:*")

| Field | Description |
|-------|-------------|
| `src_ip` | Source IP address |
| `dest_ip` | Destination IP address |
| `src_port` | Source port |
| `dest_port` | Destination port |
| `protocol` | Network protocol (TCP, UDP) |
| `bytes_in` | Bytes received |
| `bytes_out` | Bytes sent |

### Firewall Logs (sourcetype="juniper:junos:firewall")

| Field | Description |
|-------|-------------|
| `action` | Permit/Deny |
| `src_ip` | Source IP |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nebulock-Inc/agentic-threat-hunting-framework](https://github.com/Nebulock-Inc/agentic-threat-hunting-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
