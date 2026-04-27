---
trigger: always_on
description: This document outlines the complete architecture for NetRaven, a network device configuration backup system designed for deployment in customer environments. NetRaven retrieves and stores device configurations for auditing, historical tracking, and operational assurance. The system prioritizes simplicity, testability, and maintainability, while still supporting targeted concurrency for tasks like device polling.
---

## NetRaven Architecture

### Executive Summary

This document outlines the complete architecture for NetRaven, a network device configuration backup system designed for deployment in customer environments. NetRaven retrieves and stores device configurations for auditing, historical tracking, and operational assurance. The system prioritizes simplicity, testability, and maintainability, while still supporting targeted concurrency for tasks like device polling.

### System Overview

#### System Diagram (ASCII)
```
                        ┌────────────────────┐
                        │     Frontend UI     │
                        │    (Vue 3 + REST)   │
                        └────────▲───────────┘
                                 │ REST API
                                 ▼
                        ┌────────────────────┐
                        │    API Service      │
                        │     (FastAPI)       │
                        └────────▲───────────┘
                                 │
             ┌──────────────────┼────────────────────┐
             │                  │                    │
      Device │          Job CRUD│           User/Auth│
     Records │                  ▼                    ▼
         ┌───┴────┐     ┌──────────────┐      ┌──────────────┐
         │Devices │     │   Jobs Table │      │   Users/Roles│
         └───┬────┘     └──────┬───────┘      └────┬─────────┘
             │                │                    │
             ▼                ▼                    ▼
      ┌────────────┐  ┌──────────────┐      ┌──────────────┐
      │ PostgreSQL │  │  Redis Queue │      │ JWT Security │
      └────┬───────┘  └──────┬───────┘      └──────────────┘
           │                 │
           ▼                 ▼
   ┌─────────────────────────────────┐
   │        Job Scheduler            │
   │    (RQ + RQ Scheduler)          │
   └────────────┬───────────────────┘
                ▼
       ┌──────────────────────────────┐
       │ Dedicated Worker Container   │
       │ (RQ Worker, Python, Docker) │
       └────────┬────────────────────┘
                ▼
       ┌───────────────────┐
       │  Device Comm Job  │
       │   (Netmiko-based) │
       └────────┬──────────┘
                ▼
        ┌─────────────┐
        │ Network Gear│
        └─────────────┘
                │
                ▼
        ┌───────────────────────────┐
        │ Git Repo (Config Storage) │
        └───────────────────────────┘
```


NetRaven supports:

1. Retrieval of running configuration and identifying information from network devices (primarily via SSH).
2. Storing configuration snapshots in a Git repository for version control.
3. Job scheduling for one-time or recurring configuration backups.
4. REST API access for external integrations.
5. Role-based web UI for inventory management and job monitoring.

The system is installed locally using Python-based services with PostgreSQL and Redis (required for RQ and RQ Scheduler). The directory structure and service boundaries support future containerization but do not require it.

### Core Architectural Principles

- **Local Redis Required**: Redis is a core local dependency for job queuing and scheduling. All job execution flows rely on it via RQ and RQ Scheduler.
- **Synchronous-First Design**: Synchronous services simplify development and debugging.
- **Targeted Concurrency**: Uses threads only where concurrency offers clear benefits (e.g., connecting to multiple devices).
- **Modular Design**: Isolated services for API, job scheduling, communication, and logging.
- **Detailed Logging**: Per-device and per-job logs with redacted and raw entries.
- **Secure by Default**: JWT authentication and role-based access controls.

### Core Services

#### 1. API Service (FastAPI - Sync Mode)

- Exposes REST API endpoints for all system functions.
- JWT-based authentication with role enforcement (admin/user).
- CRUD for:
  - Devices
  - Device groups (tags)
  - Jobs
  - Users and roles
- Status endpoints for job monitoring.

#### 2. Device Communication Worker (now Dedicated Container)

- Executes device jobs triggered by scheduler or on demand.
- Runs as a dedicated Docker container (`worker` service) using RQ Worker.
- Connects to network devices using **Netmiko**.
- Retrieves `show running-config` and basic facts (hostname, serial number).
- Uses `ThreadPoolExecutor` for concurrent device access (default: 5).
- Reports logs and results to database.
- Picks up jobs from the Redis queue and updates job status in the database.

#### 2a. System Jobs (e.g., Reachability)
- System jobs are created automatically during system installation (e.g., reachability job).
- System jobs are associated with a default tag and credentials.
- System jobs are not user-deletable or editable.
- The worker processes system jobs just like regular jobs, but with special handling for job type and status.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ccieblogger) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
