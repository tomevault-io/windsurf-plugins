---
trigger: always_on
description: description: llm-food project: TaskStateRepository (DuckDB) for persistent batch job state management using local DuckDB.
---

---
description: llm-food project: TaskStateRepository (DuckDB) for persistent batch job state management using local DuckDB.
globs: llm_food/app.py
alwaysApply: false
---
# Chapter 7: TaskStateRepository (DuckDB)

In Chapter 6, [PDFProcessingStrategy (Synchronous)](pdfprocessingstrategy__synchronous_.mdc), we detailed the synchronous PDF processing strategies. Now, we turn to the persistence layer critical for the asynchronous batch operations discussed in [BatchJobOrchestrator](batchjoborchestrator.mdc): the `TaskStateRepository (DuckDB)`. This repository forms the persistence backbone for managing the state of asynchronous batch jobs within the `llm-food` project.

## Motivation and Purpose

The primary technical problem solved by the `TaskStateRepository` is the **reliable tracking of long-running, potentially multi-stage asynchronous operations**. When dealing with batch jobs that might involve processing hundreds of files, interacting with external APIs (like Gemini), and taking significant time to complete, it's crucial to have a durable way to store and manage the state of these operations. Without such a persistence layer, server restarts or unexpected failures could lead to loss of job progress and an inability to report status or retrieve results.

The `TaskStateRepository` utilizes a local DuckDB database to:
- Store information about overall batch jobs (`batch_jobs` table).
- Track details of Gemini API calls for PDF processing (`gemini_pdf_batch_sub_jobs` table).
- Manage the status of individual file or page-level tasks (`file_tasks` table).
- Provide functions for schema initialization, record insertion, status updates (e.g., 'pending', 'processing', 'completed', 'failed'), and querying job details.

DuckDB was chosen for its simplicity (file-based, embedded), SQL interface, and ease of integration into a Python application like `llm-food`. It provides sufficient durability and querying capabilities for the project's needs.

**Central Use Case:**
The [BatchJobOrchestrator](batchjoborchestrator.mdc) receives a request to the `/batch` endpoint with multiple files.
1.  It immediately creates a record in the `batch_jobs` table with a unique `job_id` and 'pending' status.
2.  For each PDF file group destined for Gemini, it creates a record in `gemini_pdf_batch_sub_jobs`.
3.  For each individual file (or PDF page processed by Gemini), it creates a record in the `file_tasks` table.
4.  As background tasks process these files/pages, they update the `status`, `gcs_output_markdown_uri`, or `error_message` fields in the respective `file_tasks` and `gemini_pdf_batch_sub_jobs` records.
5.  The overall progress (`overall_processed_count`, `overall_failed_count`) and status of the main job in `batch_jobs` are updated.
6.  Clients polling the `/status/{task_id}` endpoint trigger queries against these tables to retrieve the current state.
7.  Once a job is 'completed' or 'completed_with_errors', the `/batch/{task_id}` endpoint queries `file_tasks` for `gcs_output_markdown_uri` to fetch and return results.

This durable state management is critical for reliability and for providing informative feedback to users about their batch jobs.

## Core Component: Database Schema

The repository's structure is defined by its database schema, implemented using SQL `CREATE TABLE` statements within DuckDB. The primary logic for schema initialization and interaction resides in `llm_food/app.py`.

### 1. `batch_jobs` Table
Stores metadata for each overarching batch job.
- **Purpose:** Tracks the overall status and summary of a batch request initiated via the `/batch` endpoint.
- **Key Columns:**
  - `job_id` (VARCHAR, PK): Unique identifier for the batch job.
  - `output_gcs_path` (VARCHAR): The GCS path where outputs should be stored.
  - `status` (VARCHAR): Overall status (e.g., 'pending', 'processing', 'completed', 'completed_with_errors', 'failed_catastrophic').
  - `submitted_at` (TIMESTAMP): When the job was submitted.
  - `total_input_files` (INTEGER): Total number of files in the batch.
  - `overall_processed_count` (INTEGER): Count of successfully processed files.
  - `overall_failed_count` (INTEGER): Count of failed files.
  - `last_updated_at` (TIMESTAMP): Timestamp of the last update to this job record.

### 2. `gemini_pdf_batch_sub_jobs` Table
Stores details specific to batch processing of PDFs using the Google Gemini API.
- **Purpose:** Tracks a single Gemini Batch API operation, which might process images derived from multiple PDF files or many pages.
- **Key Columns:**
  - `gemini_batch_sub_job_id` (VARCHAR, PK): Unique ID for this sub-job.
  - `batch_job_id` (VARCHAR, FK): Links to the main `batch_jobs.job_id`.
  - `gemini_api_job_name` (VARCHAR): The job name returned by the Gemini API.
  - `status` (VARCHAR): Status of this sub-job (e.g., 'pending_preparation', 'submitting_to_gemini', 'JOB_STATE_SUCCEEDED', 'failed_gemini_job_JOB_STATE_FAILED').
  - `payload_gcs_uri` (VARCHAR): GCS URI of the `payload.jsonl` sent to Gemini.
  - `gemini_output_gcs_uri_prefix` (VARCHAR): GCS prefix where Gemini writes its results.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [altaidevorg/llm-food](https://github.com/altaidevorg/llm-food) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
