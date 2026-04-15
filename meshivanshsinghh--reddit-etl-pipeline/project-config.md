---
trigger: always_on
description: This is a Tesla Energy interview project demonstrating real-time data engineering skills.
---

# Reddit Energy Sentiment Pipeline - Updated Rules & Context
# Last Updated: November 21, 2024
# Interview: Monday (2 days away)
# Current Status: Core pipeline complete, adding advanced analytics

## Project Context
This is a Tesla Energy interview project demonstrating real-time data engineering skills.
The pipeline processes Reddit discussions about renewable energy to extract insights,
mirroring Tesla's need to process Powerwall telemetry data.

## Current Architecture Status

### ✅ COMPLETED & WORKING (DO NOT CHANGE):
1. Docker infrastructure (5 containers: Zookeeper, Kafka, PostgreSQL, Airflow x2)
2. Reddit ingestion pipeline with Kafka producer (reddit_kafka/producer.py)
3. Kafka consumer integrated into Airflow DAG (writes to PostgreSQL)
4. Database schema with 4 tables (posts_raw, sentiment_timeseries, daily_metrics, data_quality_metrics)
5. Sentiment analysis using VADER (utils/sentiment_analyzer.py)
6. Data quality monitoring with 4 checks (monitoring/quality_checks.py)
7. Daily analytics DAG (airflow/dags/daily_analytics_dag.py)
8. 300+ posts ingested with sentiment analyzed (57% positive, 18% negative, 25% neutral)

### 🚧 IN PROGRESS (FOCUS HERE):
1. Topic extraction & issue clustering (analytics/topic_extractor.py) - PRIORITY 1
2. Real-time anomaly detection (analytics/anomaly_detector.py) - PRIORITY 2
3. Updated dashboard to show new insights - PRIORITY 3

### ⚠️ NOT IMPLEMENTED (DON'T BUILD UNLESS ASKED):
1. Spark Streaming deployment (code exists but commented out for M1 Mac performance)
2. Cloud deployment (architecture ready, but demo runs locally)
3. Advanced ML models
4. CI/CD pipeline

## Current Data State
- 300 posts in posts_raw table
- 300 sentiment records in sentiment_timeseries
- 2 daily metrics records
- 5 quality check records
- All running on localhost (Docker containers)

## Technical Decisions Made (DO NOT CHANGE):

### Database:
- PostgreSQL with direct connections (not PostgresHook due to auth issues)
- PRIMARY KEY on posts_raw.id for deduplication
- JSONB for flexible schema (details in data_quality_metrics, top_posts in daily_metrics)
- Indexes on timestamp fields for time-series queries

### Kafka:
- Topic: reddit_raw_posts
- Manual offset commits for reliability
- No partitioning (single broker, small scale)
- Consumer integrated in Airflow DAG (not separate service)

### Airflow:
- LocalExecutor (not CeleryExecutor)
- Manual triggers for all DAGs (schedule_interval=None except daily_analytics)
- Direct psycopg2 connections (not using Airflow Connections)
- Environment variables: POSTGRES_HOST, POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD

### Sentiment Analysis:
- VADER (not Hugging Face or custom models)
- Runs as standalone script with exported env vars
- Not integrated into streaming pipeline (manual execution)
- Processes all posts at once (batch mode)

## Known Issues & Accepted Trade-offs:

1. **Duplicate posts from /hot endpoint:** Accepted - PRIMARY KEY handles it
2. **Manual sentiment analysis:** Accepted - works for demo scale
3. **No Spark deployment:** Accepted - code ready but not needed for demo
4. **Basic HTML dashboard:** Accepted - functional but needs improvement
5. **Limited post volume (300 posts):** Accepted - Reddit API limitation for demo

## Code Style & Patterns to Follow:

### Python:
- Type hints for function parameters and returns
- Docstrings for all functions (Google style)
- Error handling with try-except and logging
- Environment variables with os.getenv() and defaults
- Use existing logger: `from utils.logger import setup_logger`

### SQL:
- Use parameterized queries (psycopg2 %s placeholders)
- Include ON CONFLICT clauses for idempotency
- Add indexes for all frequently queried fields
- Use JSONB for flexible/nested data

### Airflow DAGs:
- default_args with retries, retry_delay, execution_timeout
- Manual trigger (schedule_interval=None) unless explicitly scheduled
- Tags for categorization
- Task IDs in snake_case
- Use PythonOperator for Python code (not BashOperator with python -c)

### File Structure:
- Place analytics code in analytics/ directory
- Place monitoring code in monitoring/ directory
- Place utility code in utils/ directory
- Airflow DAGs go in airflow/dags/
- SQL migrations in sql/ directory

## Next Features to Implement:

### 1. Topic Extraction & Issue Clustering (PRIORITY 1)
**Files to Create:**
- analytics/topic_extractor.py
- airflow/dags/topic_extraction_dag.py

**New Tables:**
```sql
CREATE TABLE topics (
    id SERIAL PRIMARY KEY,
    subreddit VARCHAR(100),
    keyword VARCHAR(100),
    score FLOAT,
    post_count INTEGER,
    avg_sentiment FLOAT,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE issue_clusters (
    id SERIAL PRIMARY KEY,
    cluster_id INTEGER,
    cluster_name VARCHAR(200),
    post_ids TEXT[],
    keywords TEXT[],
    avg_sentiment FLOAT,
    post_count INTEGER,
    severity VARCHAR(20),
    created_at TIMESTAMP DEFAULT NOW()
);
```

**Implementation Requirements:**
- Use scikit-learn TfidfVectorizer for keyword extraction
- Use KMeans clustering (n_clusters=5-10)
- Process all posts from posts_raw table
- Assign severity based on negative sentiment + post count
- Store results in both tables
- Create Airflow DAG to run daily or on-demand

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/meshivanshsinghh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
