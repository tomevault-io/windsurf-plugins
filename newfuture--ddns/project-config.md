---
trigger: always_on
description: > **Comprehensive guide for AI agents working on the DDNS (Dynamic DNS) project**
---

# AGENTS.md - AI Agent Guide for DDNS Project

> **Comprehensive guide for AI agents working on the DDNS (Dynamic DNS) project**

## Table of Contents

1. [Project Overview](#project-overview)
2. [Project Architecture](#project-architecture)
3. [Getting Started](#getting-started)
4. [Development Guide](#development-guide)
5. [Testing & Validation](#testing--validation)
6. [Troubleshooting](#troubleshooting)
7. [Best Practices](#best-practices)

---

## Project Overview

### What is DDNS?

DDNS is a Python-based Dynamic DNS client that automatically updates DNS records to match the current IP address. It supports:

- **Multiple DNS Providers**: 15+ providers including Cloudflare, DNSPod, AliDNS, etc.
- **Dual Stack**: IPv4 and IPv6 support
- **Multiple Platforms**: Docker, binary executables, pip installation, and source code
- **Flexible Configuration**: Command-line arguments, JSON files, and environment variables
- **Advanced Features**: Multi-domain support, HTTP proxy, caching, scheduled tasks

### Key Technologies

- **Language**: Python (2.7+ and 3.x compatible)
- **Testing**: unittest (default) and pytest (optional)
- **Linting/Formatting**: ruff
- **CI/CD**: GitHub Actions
- **Containerization**: Docker (multi-architecture support)
- **Packaging**: PyPI, Nuitka (for binaries)

### Project Status

- **License**: MIT
- **Python Versions**: 2.7, 3.6, 3.7, 3.8, 3.9, 3.10, 3.11, 3.12, 3.13, 3.14
- **Platforms**: Windows, Linux, macOS
- **Architectures**: amd64, arm64, arm/v7, arm/v6, 386, ppc64le, riscv64, s390x

---

## Project Architecture

### Directory Structure

Here is the folder and file structure for the DDNS project.

**Format:** `<TAB depth>{filename}:<TAB>{description}`

```text
.github/:	GitHub configuration
	workflows/:	CI/CD workflows (build, publish, test)
	instructions/:	Agent instructions (python.instructions.md)
	copilot-instructions.md:	GitHub Copilot instructions

ddns/:	Main application code
	__init__.py:	Package initialization and version info
	__main__.py:	Entry point for module execution
	cache.py:	Cache management
	ip.py:	IP address detection logic

	config/:	Configuration management
		__init__.py
		cli.py:	Command-line argument parsing
		config.py:	Configuration loading and merging
		env.py:	Environment variable parsing
		file.py:	JSON file configuration

	provider/:	DNS provider implementations
		__init__.py:	Provider registry
		_base.py:	Abstract base classes (SimpleProvider, BaseProvider)
		_signature.py:	HMAC signature utilities
		alidns.py:	Alibaba Cloud DNS
		aliesa.py:	Alibaba Cloud ESA
		callback.py:	Custom webhook callbacks
		cloudflare.py:	Cloudflare DNS
		cloudns.py:	ClouDNS
		debug.py:	Debug provider
		dnscom.py:	DNS.COM
		dnspod.py:	DNSPod (China)
		dnspod_com.py:	DNSPod International
		edgeone.py:	Tencent EdgeOne
		edgeone_dns.py:	Tencent EdgeOne DNS
		he.py:	Hurricane Electric
		huaweidns.py:	Huawei Cloud DNS
		namesilo.py:	NameSilo
		noip.py:	No-IP
		tencentcloud.py:	Tencent Cloud DNS
		west.py:	West.cn DNS

	scheduler/:	Task scheduling implementations
		__init__.py
		_base.py:	Base scheduler class
		cron.py:	Cron-based scheduler (Linux/macOS)
		launchd.py:	macOS launchd scheduler
		schtasks.py:	Windows Task Scheduler
		systemd.py:	Linux systemd timer

	util/:	Utility modules
		__init__.py
		comment.py:	Comment handling
		fileio.py:	File I/O operations
		http.py:	HTTP client with proxy support
		try_run.py:	Safe command execution

tests/:	Unit tests
	__init__.py:	Test initialization (path setup)
	base_test.py:	Shared test utilities and base classes
	README.md:	Testing documentation
	config/:	Test configuration files
	scripts/:	Test helper scripts
	test_cache.py:	Cache tests
	test_config_*.py:	Configuration tests
	test_ip.py:	IP detection tests
	test_provider_*.py:	Provider-specific tests
	test_scheduler_*.py:	Scheduler tests
	test_util_*.py:	Utility tests

docs/:	Documentation (VitePress-based)
	.vitepress/:	VitePress configuration and theme
	
	config/:	Configuration documentation (Chinese)
		cli.md:	CLI usage guide
		env.md:	Environment variables guide
		json.md:	JSON configuration guide

	dev/:	Developer guides (Chinese)
		provider.md:	Provider development guide
		config.md:	Configuration system design

	providers/:	Provider-specific documentation (Chinese)
		README.md:	Provider list and overview
		51dns.md:	51DNS provider guide
		alidns.md:	Alibaba Cloud DNS guide
		aliesa.md:	Alibaba Cloud ESA guide
		callback.md:	Custom webhook callbacks guide
		cloudflare.md:	Cloudflare DNS guide
		cloudns.md:	ClouDNS guide
		debug.md:	Debug provider guide
		dnscom.md:	DNS.COM provider guide
		dnspod.md:	DNSPod (China) guide
		dnspod_com.md:	DNSPod International guide
		edgeone.md:	Tencent EdgeOne guide
		edgeone_dns.md:	Tencent EdgeOne DNS guide
		he.md:	Hurricane Electric guide
		huaweidns.md:	Huawei Cloud DNS guide
		namesilo.md:	NameSilo guide
		noip.md:	No-IP guide
		tencentcloud.md:	Tencent Cloud DNS guide
		west.md:	West.cn DNS guide

	en/:	English documentation
		config/:	English configuration guides (mirrors config/)
		dev/:	English developer guides (mirrors dev/)
		providers/:	English provider guides (mirrors providers/)
		docker.md:	Docker documentation
		install.md:	Installation guide

	public/:	Public static assets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NewFuture/DDNS](https://github.com/NewFuture/DDNS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
