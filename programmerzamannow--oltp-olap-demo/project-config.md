---
trigger: always_on
description: Demo edukasi sinkronisasi real-time dari OLTP ke OLAP. Repo ini dibangun sebagai demo terbuka di GitHub: `git@github.com:ProgrammerZamanNow/oltp-olap-demo.git`.
---

# Project Context — OLTP → OLAP CDC Demo

Demo edukasi sinkronisasi real-time dari OLTP ke OLAP. Repo ini dibangun sebagai demo terbuka di GitHub: `git@github.com:ProgrammerZamanNow/oltp-olap-demo.git`.

## Preferensi Kerja

- Bahasa diskusi: **Bahasa Indonesia**.
- Backend code: **Spring Boot + Java 21**. Bukan Python/Node/Go.
- Container runtime: **Podman**, bukan Docker. Compose file diberi nama `podman-compose.yml`. Image selalu pakai prefix registry eksplisit (`docker.io/...`, `quay.io/...`) karena Podman tidak punya default registry.
- Git: **tidak boleh** ada `Co-Authored-By: Claude` atau mention "Generated with Claude Code" di commit message / PR body.

## Arsitektur

```
Spring Boot → PostgreSQL → Debezium (Kafka Connect) → Apache Kafka → ClickHouse
generator     (OLTP)       baca WAL via pgoutput       shop.public.*   Kafka engine + MV
                                                                       → ReplacingMergeTree
```

5 komponen, semuanya jalan sebagai container via `podman-compose.yml`. Generator di-build dari `data-generator/Dockerfile` (multi-stage maven → jre 21). Untuk fast iteration tanpa rebuild image, masih bisa run dari host pakai `make generator-host` — `application.yml` default-nya connect ke `localhost:15432`, sedangkan service compose override via `SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/shop`.

### Keputusan Teknis Penting

| Pilihan | Alasan |
|---------|--------|
| `apache/kafka:3.8.0` (bukan `bitnami/kafka:3.8`) | Tag bitnami sudah dihapus dari Docker Hub — manifest unknown. Env var beda: pakai `KAFKA_*` tanpa prefix `KAFKA_CFG_`. |
| Kafka KRaft mode (no Zookeeper) | Lebih ringan untuk demo. CLUSTER_ID di-hardcode supaya consistent restart. |
| `wal_level=logical` + `REPLICA IDENTITY FULL` | Wajib untuk Debezium pgoutput CDC. FULL supaya UPDATE/DELETE event bawa before-state lengkap (cost: WAL lebih besar, OK untuk demo). |
| `plugin.name=pgoutput` | Native ke Postgres modern (≥10), tidak butuh extension tambahan seperti wal2json. |
| `unwrap` SMT (ExtractNewRecordState) | Flatten payload Debezium jadi field-field tabel langsung. `delete.handling.mode=rewrite` → tambah kolom `__deleted` ('true'/'false') alih-alih tombstone. ClickHouse JSONEachRow gampang parse. |
| `decimal.handling.mode=double` | DECIMAL Postgres → JSON number. ClickHouse cast ke Decimal64 di MV (lossy tapi cukup untuk demo). Alternatif: `string` + parse manual untuk presisi penuh. |
| `time.precision.mode=connect` | TIDAK menghasilkan Int64 millis seperti dugaan awal — dengan `JsonConverter` schemaless tetap di-serialize sebagai **ISO 8601 string**. Karena itu Kafka engine table di ClickHouse pakai `String` + `parseDateTime64BestEffortOrZero(..., 3)`. |
| `ReplacingMergeTree(updated_at)` | Versi terbaru per PK menang setelah background merge. Query ad-hoc pakai `FINAL` untuk dedup at query-time. |
| Soft delete via `is_deleted` UInt8 | Bukan DELETE FROM. Filter `WHERE is_deleted = 0` saat query. Konsisten dengan event-based CDC. |
| Dua MV per `kafka_orders` → `orders` (ReplacingMergeTree, current state) + `orders_events` (MergeTree, full history) | `windowFunnel` butuh history event status. ReplacingMergeTree dedupe per `(created_at, id)` jadi history hilang setelah merge. Tabel `orders_events` ORDER BY `(id, updated_at)` jadi tiap event row unik, tidak ke-dedup. ClickHouse fanout: dua MV pada Kafka engine yang sama tetap dapat semua event. |
| Sintaks `FINAL` di ClickHouse | Harus **setelah alias**: `FROM tbl AS x FINAL`, BUKAN `FROM tbl FINAL AS x`. |
| Port Postgres host = `15432` (bukan 5432 / 5433) | User punya Postgres lokal di 5432, dan port dev umum lain (5433) juga sering dipakai. Pakai 15432 yang very uncommon untuk hindari bentrok. Container internal tetap 5432. Generator's `application.yml` connect ke `jdbc:postgresql://localhost:15432/shop`. Service compose pakai DNS internal `postgres:5432` via env `SPRING_DATASOURCE_URL`. |
| Tidak ada volume Kafka di compose | Sengaja — restart `down` akan wipe topic & connector configs. Demo ulang dari snapshot lebih bersih. (Untuk production: tambahkan volume.) |

## Setup & Run

```bash
# Pastikan podman machine sudah running dengan memory cukup (minimal 4 GiB, ideal 6 GiB)
podman machine set --memory 6144 --cpus 4
podman machine start

# Stack up (build image generator + jalankan 5 container)
make up         # podman compose up -d

# Tunggu ~30 detik, register Debezium connector
make register   # POST debezium/postgres-connector.json ke localhost:8083

# Cek connector RUNNING
make status

# Generator log (sudah auto-jalan via compose)
make generator-logs

# Inspect data OLTP via REST API generator (port 8080)
curl -s http://localhost:8080/api/orders | jq
# query params: ?page=0&size=100&sort=createdAt,desc — default size=100, sort=createdAt DESC

# Time-series metrics endpoints (ClickHouse-backed)
curl -s http://localhost:8080/api/metrics/windows           # 8 metric single-pass
curl -s "http://localhost:8080/api/metrics/throughput?withFill=true&minutes=60"
curl -s "http://localhost:8080/api/metrics/velocity?minutes=30"
curl -s "http://localhost:8080/api/metrics/anomaly?minutes=60"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ProgrammerZamanNow/oltp-olap-demo](https://github.com/ProgrammerZamanNow/oltp-olap-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
