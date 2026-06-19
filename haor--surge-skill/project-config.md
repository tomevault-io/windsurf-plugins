---
trigger: always_on
description: >
---


# Surge Documentation Skill

## How to use

1. Match the user's question to one or more topics in the **Routing Table** below.
2. Use the `Read` tool to load the matched file(s). All paths are relative to this skill's directory (`.claude/skills/surge/`). Prepend the project root to form absolute paths (e.g., `{project_root}/.claude/skills/surge/docs/manual/rule/domain-based.md`). Read at most **3 files** per query.
3. Synthesize an answer from the loaded content. Always **cite the source file path**.
4. If the documentation does not cover the question, say so explicitly.

## Language rules

- The manual (`docs/manual/`) is English-only.
- The knowledge base (`docs/kb/`) has both `en/` and `zh/` versions.
- When the user asks in **Chinese**, prefer `docs/kb/zh/` files for KB topics.
- When the user asks in **English**, prefer `docs/kb/en/` files for KB topics.
- For manual topics, always use `docs/manual/` regardless of language, then answer in the user's language.

## Cross-topic dependencies

Some queries require reading files from multiple sections. Key relationships:

- **MITM is prerequisite for HTTPS processing**: URL rewrite, header rewrite, body rewrite, and HTTP scripting on HTTPS traffic all require MITM to be enabled. If the user asks "why doesn't my rewrite work on HTTPS", read `mitm.md` alongside the relevant rewrite doc.
- **Enhanced mode is prerequisite for gateway mode**: Gateway setup requires enhanced mode (TUN/VIF). Read `enhanced-mode.md` alongside `gateway.md`.
- **IP rules interact with DNS**: IP-based rules trigger DNS resolution unless `no-resolve` is specified. If the user asks about DNS + rule interaction, read `kb/technotes/dns.md` alongside `rule/ip-based.md`.
- **Module authoring spans multiple topics**: Writing a module typically requires `module.md` + one of the rewrite/scripting docs + `mitm.md`.
- **Proxy subscription + policy group**: Setting up proxy subscriptions usually involves `external-proxy.md` + a policy group doc (e.g., `url-test.md` or `select.md`) + `policy-including.md`.
- **Scripting common API is shared**: `scripting/common.md` contains `$httpClient`, `$notification`, `$persistentStore` used by all script types. Read it alongside specific script docs when the user asks about scripting APIs.

## Routing Table

All paths below are relative to this skill's directory (`.claude/skills/surge/`).

### Getting Started & Overview

| Topic | File |
|-------|------|
| Surge components overview (架构) | `docs/manual/overview/components.md` |
| Configuration file format (配置文件, [Proxy], [Rule], [General], #!include, Linked Profile, #!REQUIREMENT) | `docs/manual/overview/configuration.md` |
| Understanding Surge (EN) | `docs/manual/book/understanding-surge/en.md` |
| Understanding Surge (CN) | `docs/manual/book/understanding-surge/cn.md` |
| Manual index | `docs/manual/index.md` |

### Rules (规则, 分流)

| Topic | File |
|-------|------|
| Rules overview (规则概述) | `docs/manual/rule.md` |
| Domain-based rules (DOMAIN, DOMAIN-SUFFIX, DOMAIN-KEYWORD, DOMAIN-SET, DOMAIN-WILDCARD) | `docs/manual/rule/domain-based.md` |
| IP-based rules (IP-CIDR, IP-CIDR6, GEOIP, IP-ASN, no-resolve) | `docs/manual/rule/ip-based.md` |
| HTTP rules (USER-AGENT, URL-REGEX, HEADER) | `docs/manual/rule/http.md` |
| Process rules (PROCESS-NAME, macOS) | `docs/manual/rule/process.md` |
| Subnet rules (SUBNET, SSID) | `docs/manual/rule/subnet.md` |
| Logical rules (AND, OR, NOT) | `docs/manual/rule/logical-rule.md` |
| Miscellaneous rules (DEST-PORT, IN-PORT, SRC-IP, PROTOCOL, SCRIPT) | `docs/manual/rule/misc-rule.md` |
| Rule sets (RULE-SET, external rule files, Inline Ruleset, extended-matching, 规则集) | `docs/manual/rule/ruleset.md` |
| FINAL rule | `docs/manual/rule/final.md` |

### Policies (代理, 策略)

| Topic | File |
|-------|------|
| Policies overview (策略概述, [Proxy]) | `docs/manual/policy.md` |
| Built-in policies (DIRECT, REJECT, REJECT-DROP, REJECT-TINYGIF, REJECT-NO-DROP, CELLULAR, HYBRID, NO-HYBRID) | `docs/manual/policy/built-in.md` |
| Proxy protocols (Shadowsocks/SS, Snell, SOCKS5, SOCKS5-TLS, HTTP/HTTPS proxy, Trojan, VMess, TUIC, Hysteria 2, AnyTLS, Shadow TLS, proxy chain/underlying-proxy, obfs, WebSocket, UDP relay, port-hopping, SNI, [Keystore]) | `docs/manual/policy/proxy.md` |
| Policy parameters (TFO, ECN, MPTCP, test-url, test-timeout, block-quic) | `docs/manual/policy/parameters.md` |
| External proxy provider (机场订阅, proxy subscription, proxy-provider, filter) | `docs/manual/policy/external-proxy.md` |
| REJECT policy behavior (REJECT variants deep-dive) | `docs/manual/policy/reject.md` |
| SSH proxy (SSH 代理, SSH tunnel) | `docs/manual/policy/ssh.md` |
| WireGuard proxy (WG, [WireGuard *], DSCP, ECN, peer, private-key) | `docs/manual/policy/wireguard.md` |

### Policy Groups (策略组, [Proxy Group])

| Topic | File |
|-------|------|
| Policy groups overview (策略组概述) | `docs/manual/policy-group/group.md` |
| Select group (手动选择, manual selection) | `docs/manual/policy-group/select.md` |
| URL-Test group (自动测速, auto best latency) | `docs/manual/policy-group/url-test.md` |
| Fallback group (故障切换, 备用) | `docs/manual/policy-group/fallback.md` |
| Load-Balance group (负载均衡) | `docs/manual/policy-group/load-balance.md` |
| Subnet group (SSID-based) | `docs/manual/policy-group/subnet.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Haor/surge_skill](https://github.com/Haor/surge_skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
