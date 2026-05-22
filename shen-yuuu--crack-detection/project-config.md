---
trigger: always_on
description: This is a **road crack detection system** using deep learning (ConvNeXt + UPerNet) with a **Java + Python hybrid microservice architecture**. The project spans three major components:
---

# AI Coding Agent Instructions for Crack Detection System

## Project Overview

This is a **road crack detection system** using deep learning (ConvNeXt + UPerNet) with a **Java + Python hybrid microservice architecture**. The project spans three major components:

- **Python AI Module** (`python-inference/`): Deep learning models, data processing, model training/inference
- **Java Backend** (`cloud-backend/`): 7 microservices (auth, gateway, dataset, inference, visual, report, common)
- **Vue.js Frontend** (`cloud-frontend/`): Modern web UI with Element Plus components

## Architecture Patterns

### Layered Architecture
```
User Layer (Vue 3) 
  ↓ HTTPS
Gateway Layer (Spring Cloud Gateway, port 8080)
  ↓ HTTP
Business Services (7 Java microservices, ports 8081-8085)
  ↓ HTTP/Feign
Python AI Service (FastAPI, port 8090)
  ↓
Data Storage (MySQL, Redis, MinIO, RabbitMQ)
```

### Key Design Decisions

1. **Java for Business Logic**: Spring Boot microservices handle user auth, data management, task orchestration, report generation
2. **Python for AI**: FastAPI service provides model inference, kept separate for independent scaling
3. **Message Queue Pattern**: RabbitMQ for async tasks (preprocessing, batch detection)
4. **Feature Extraction**: Multiple models - ConvNeXt Backbone, UPerNet Decoder, CBAM Attention, Strip Pooling for elongated cracks
5. **API Gateway**: Centralized auth (JWT), rate limiting, request routing

## Critical Files & Entry Points

### Python AI Module
- **Model Definition**: `python-inference/models/convnext_upernet.py` - ConvNeXt backbone + UPerNet decoder with edge detection branch
- **Data Loading**: `python-inference/dataset/data_loader.py` - COCO/VOC/YOLO format conversion, LMDB caching, 10x I/O speedup
- **Training**: `python-inference/training/trainer.py` - Mixed precision (AMP), EMA, SWA, early stopping
- **Inference**: `python-inference/inference/sliding_window.py` - Sliding window for arbitrary-sized images, TTA for 2-3% mIoU boost
- **API Server**: `python-inference/inference/api_server.py` - FastAPI endpoints for model inference
- **Config**: `python-inference/configs/train_config.yaml` - Training hyperparameters

**Key Performance**: mIoU 81.5%, F1 86.7%, 28 FPS on 512×512 images

### Java Backend Structure
```
cloud-backend/
├── cloud-common/          # Entities, DTOs, constants, utilities
├── cloud-auth/            # JWT token generation/validation (port 8081)
├── cloud-gateway/         # API routing, auth filter, rate limiting (port 8080)
├── cloud-dataset/         # Dataset/image CRUD, MinIO integration (port 8082)
├── cloud-inference/       # Feign client to Python service, job management (port 8083)
├── cloud-visual/          # Image overlay, heatmap generation (port 8084)
└── cloud-report/          # PDF/Excel generation via iText/EasyExcel (port 8085)
```

**Database**: MySQL with 6 core tables (users, datasets, images, detection_jobs, detection_results, reports)

### Frontend
- **Router**: `cloud-frontend/src/router/index.js` - Auth guards, role-based access control
- **API Integration**: `cloud-frontend/src/api/` - Axios wrapper with JWT auto-refresh
- **State Management**: `cloud-frontend/src/stores/user.js` - Pinia store
- **Pages**: 9 core views (login, dashboard, detection, dataset, history, report, profile, 404)

## Development Workflows

### Python Model Development
```bash
# Prepare multi-source datasets (COCO/VOC/YOLO → PNG masks)
cd python-inference
python scripts/prepare_datasets.py --source ../datasets --output ../data/processed

# Verify data quality and augmentations
python scripts/visualize_dataset.py --data-root ../data/processed --mode check

# Train with mixed precision, EMA, SWA
python train.py --config configs/train_config.yaml

# Export to ONNX for production
python export_onnx.py --model checkpoints/best.pth --output model.onnx

# Start inference API (used by Java services)
python inference/api_server.py
```

### Java Backend Development
```bash
# Start infrastructure (MySQL, Redis, MinIO, RabbitMQ, Nacos)
cd docker && docker-compose up -d

# Build all services
cd cloud-backend && mvn clean package

# Start individual services (they auto-register with Nacos)
java -jar cloud-gateway/target/cloud-gateway-1.0.0.jar
# Then other services discover gateway via Nacos registry

# Health checks
curl http://localhost:8080/api/v1/auth/health
curl http://localhost:8090/api/v1/health  # Python service
```

### Frontend Development
```bash
# Install dependencies
cd cloud-frontend && npm install

# Development server with hot reload
npm run dev

# Build for production
npm run build

# Test API endpoints (assumes backend running on localhost:8080)
```

## Code Conventions & Patterns

### Java Backend Conventions
- **Entity Naming**: Use `@TableName("table_name")` annotations (plural form: users, datasets, images)
- **DTO Naming**: Use suffixes like `Request`, `Response`, `VO`, `DTO`
- **API Endpoints**: Follow `/api/v1/{service}/{resource}` pattern (e.g., `/api/v1/dataset/images`)
- **Service Layer**: Each module has `Service` → `Mapper` (MyBatis Plus) → Database flow

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shen-Yuuu/crack-detection](https://github.com/Shen-Yuuu/crack-detection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
