---
trigger: always_on
description: **Last Updated**: 2025-11-21
---

# CLAUDE.md - Stridetastic Server Complete Codebase Documentation

**Last Updated**: 2025-11-21  
**Version**: 1.0  
**Total Python LOC**: ~11,747  
**Total TypeScript Files**: 64  
**Architecture**: Django + Next.js + TimescaleDB + Celery + Redis

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Technology Stack](#technology-stack)
4. [Project Structure](#project-structure)
5. [Core Components](#core-components)
6. [Data Models](#data-models)
7. [Services & Business Logic](#services--business-logic)
8. [API Endpoints](#api-endpoints)
9. [Frontend Architecture](#frontend-architecture)
10. [Deployment](#deployment)
11. [Security Considerations](#security-considerations)
12. [Development Guide](#development-guide)
13. [Testing](#testing)
14. [Future Roadmap](#future-roadmap)

---

## Project Overview

**Stridetastic Server** is a comprehensive security research tool for analyzing and testing Meshtastic mesh radio networks. It provides both passive network monitoring (sniffing) and active security testing (publishing) capabilities.

### Purpose
- **Network Analysis**: Capture and analyze Meshtastic mesh network traffic
- **Security Research**: Test mesh network resilience through controlled publishing
- **Penetration Testing**: Identify vulnerabilities in mesh communications
- **Education**: Learn about mesh networking protocols and security

### Key Features
- Multi-interface support (MQTT, Serial/RF, TCP/Network)
- Real-time packet capture with PCAP-NG export
- AES and PKI encryption/decryption
- Publishing: NodeInfo, Position, Text Messages, Traceroutes
- Reactive and periodic publishing modes
- Virtual node management
- Real-time network visualization
- TimescaleDB for time-series analysis
- Grafana dashboards for metrics

---

## Architecture

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        USER LAYER                            │
├─────────────────────────────────────────────────────────────┤
│  Web Frontend (Next.js)  │  Grafana Dashboards  │  Admin   │
│  Port 3000               │  Port 3001           │  Panel   │
└──────────────┬───────────┴──────────────────────┴──────────┘
               │
┌──────────────▼──────────────────────────────────────────────┐
│                     API LAYER (Django)                       │
│  Port 8000 - Django-Ninja REST API + JWT Auth               │
├─────────────────────────────────────────────────────────────┤
│  Controllers: Auth, Node, Channel, Publisher, Capture, etc.   │
└──────────────┬──────────────────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────────────────┐
│                    SERVICE LAYER                             │
├─────────────────────────────────────────────────────────────┤
│  ServiceManager (Orchestration)                              │
│  ├── SnifferService    (Passive capture)                     │
│  ├── PublisherService    (Active injection)                    │
│  ├── CaptureService    (PCAP writing)                        │
│  ├── PKIService        (Encryption)                          │
│  └── VirtualNodeService (Node management)                    │
└──────────────┬──────────────────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────────────────┐
│                  INTERFACE LAYER                             │
├─────────────────────────────────────────────────────────────┤
│  MqttInterface  SerialInterface  TcpInterface  WebSocketInterface │
│  (Multiple instances supported per type)                     │
└──────────────┬──────────────────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────────────────┐
│                  INGESTION LAYER                             │
├─────────────────────────────────────────────────────────────┤
│  Dispatcher → PacketHandler → Payload Processors             │
│  ├── NodeInfo Handler                                        │
│  ├── Position Handler                                        │
│  ├── Telemetry Handler                                       │
│  ├── NeighborInfo Handler                                    │
│  ├── RouteDiscovery Handler                                  │
│  └── Routing Handler                                         │
└──────────────┬──────────────────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────────────────┐
│                BACKGROUND TASKS (Celery)                     │
├─────────────────────────────────────────────────────────────┤
│  Workers:                                                     │
│  ├── Sniffer Tasks     (Long-running capture)                │
│  ├── Publisher Tasks     (Periodic injection)                  │
│  └── Capture Tasks     (PCAP management)                     │
│                                                              │
│  Beat Scheduler: process_periodic_publish_jobs (30s)          │
└──────────────┬──────────────────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────────────────┐
│                   DATA LAYER                                 │
├─────────────────────────────────────────────────────────────┤

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0wulf/stridetastic](https://github.com/0wulf/stridetastic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
