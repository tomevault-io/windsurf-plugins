---
trigger: always_on
description: This document contains important context and lessons learned for any Gemini agent working on this codebase. Adhering to these guidelines will ensure smoother and more efficient development.
---

# Gemini Agent Instructions for the Harbinger Project

This document contains important context and lessons learned for any Gemini agent working on this codebase. Adhering to these guidelines will ensure smoother and more efficient development.

## 1. Getting Started

This section provides a step-by-step guide to get the project up and running for development.

### 1.1. Virtual Environment

The project uses a Python virtual environment. The convention is to locate it at `harbinger/venv/`. You might see other `venv` directories in the project root (`venv/`, `venv2/`, etc.), but `harbinger/venv/` is the correct one to use.

To create and activate the virtual environment:

```bash
python3 -m venv harbinger/venv
source harbinger/venv/bin/activate
```

### 1.2. Dependencies

The project's dependencies are managed in `requirements.txt` and `requirements_dev.txt`.

To install all necessary dependencies, run the following commands from the `harbinger` directory:

```bash
pip install -r requirements.txt
pip install -r requirements_dev.txt
```

### 1.3. Database Migrations

The project uses Alembic for database migrations. After setting up the environment and installing dependencies, you need to run the database migrations to set up the database schema.

The `pyproject.toml` file contains a script for this:

```bash
harbinger_migrate
```

### 1.4. Running the Application

The application consists of a backend (FastAPI) and a frontend (Quasar).

*   **To run the backend:**

    ```bash
    task uvicorn/reload
    ```

*   **To run the frontend:**

    ```bash
    task frontend
    ```

## 2. Codebase Overview

This section provides a high-level overview of the project's structure and where to find key features.

*   `harbinger/src/harbinger/`: This is the main application directory.
    *   `api/`: Contains the FastAPI application.
        *   `v1/endpoints/`: This is where all the API endpoints are defined. Each file corresponds to a specific feature (e.g., `files.py`, `users.py`).
    *   `models/`: Contains the SQLAlchemy database models. Each file represents a database table.
    *   `schemas/`: Contains the Pydantic schemas used for data validation and serialization. These are used in the API endpoints to define the shape of the request and response bodies.
    *   `crud/`: Contains the Create, Read, Update, Delete (CRUD) logic for the database models. This is where the database operations are defined.
    *   `worker/`: Contains the Temporal.io background worker, workflows, and activities.
    *   `connectors/`: Contains the code for interacting with external systems, such as C2 frameworks.
    *   `config/`: Contains the application's configuration and dependencies. `app.py` is the main entry point for the FastAPI application, and `dependencies.py` defines the dependencies used in the API endpoints.
    *   `scripts/`: Contains various scripts for development and maintenance, such as `generate_boilerplate.py` for creating new models, schemas, and CRUD files.
*   `harbinger/interface/`: Contains the Quasar frontend application.
*   `go/`: Contains Go services used by the application.
*   `proto/`: Contains the Protocol Buffer definitions used for communication between services.

## 3. Configuration

The application is configured primarily through environment variables. Key configuration variables include:

*   `pg_dsn`: The PostgreSQL database connection string.
*   `redis_dsn`: The Redis connection string.
*   `minio_access_key`: The MinIO access key.
*   `minio_secret_key`: The MinIO secret key.
*   `minio_host`: The MinIO host.
*   `minio_default_bucket`: The default MinIO bucket.
*   `temporal_host`: The Temporal host.

## 4. API Documentation

The project uses FastAPI, which automatically generates API documentation. You can access the Swagger UI at `http://localhost:8000/docs` and the ReDoc documentation at `http://localhost:8000/redoc` when the backend is running.

## 5. Go Services & Protocol Buffers

The project includes Go services located in the `go/` directory. These services communicate with the Python backend using gRPC. The Protocol Buffer definitions are located in the `proto/` directory.

To compile the protocol buffers, run the following command:

```bash
task protoc
```

## 6. Background Worker & Connectors

The project uses a background worker to process long-running tasks. The worker is built using the `temporalio` library and is defined in `harbinger/src/harbinger/worker/run_worker.py`.

The worker is responsible for:

*   Running playbooks
*   Processing files
*   Interacting with C2 servers
*   Generating suggestions
*   And more.

The worker is divided into two main task queues:

*   `WORKER_TASK_QUEUE`: For general tasks.
*   `FILE_PROCESSING_TASK_QUEUE`: For file processing tasks.

The project also has a system of connectors for interacting with external systems. The main connector is the `C2ConnectorWorkflow`, which is responsible for interacting with C2 servers. The `connectors` directory contains the code for the various connectors.

## 7. Project Structure: Avoid Monolithic Files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mandiant/harbinger](https://github.com/mandiant/harbinger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
