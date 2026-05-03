---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md (v1.0.7)

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요
도메인 범용화된 완벽한 오픈소스 RAG 시스템. 2026년 기준 가장 진보된 RAG 기술들을 하나의 표준 파이프라인으로 통합한 엔터프라이즈급 솔루션입니다.

- **버전**: 1.0.7
- **상태**: ✅ **2,200+ 테스트 통과**, ✅ **보안 완비**, ✅ **DI 패턴 완성**, ✅ **Streaming API**, ✅ **WebSocket**
- **주요 개선**: Reranker 확장 - Cohere, Local(sentence-transformers), OpenRouter 추가 (v1.2.1)

## 🚀 시작하기

두 가지 실행 방법을 제공합니다:

|  | Full API 서버 (`make start`) | CLI 챗봇 (`make easy-start`) |
|---|---|---|
| **Docker** | 필요 | 불필요 |
| **Vector DB** | Weaviate (하이브리드 검색) | ChromaDB (로컬 파일) |
| **인터페이스** | REST API + Swagger UI | 터미널 CLI |
| **LLM** | 5종 (Gemini, OpenAI, Claude, OpenRouter, Ollama) | Gemini / OpenRouter / Ollama |
| **용도** | 프로덕션, API 통합, 팀 개발 | 학습, 체험, 빠른 PoC |

### 방법 A: Full API 서버 (Docker)

```bash
git clone https://github.com/youngouk/OneRAG.git
cd OneRAG && uv sync
cp quickstart/.env.quickstart .env  # GOOGLE_API_KEY만 설정
make start                           # → http://localhost:8000/docs
```

### 방법 B: 로컬 CLI 챗봇 (Docker 불필요)

```bash
git clone https://github.com/youngouk/OneRAG.git
cd OneRAG && uv sync
make easy-start                      # → 터미널에서 바로 대화
```

API 키 없이도 검색은 작동합니다. AI 답변 생성을 사용하려면:
- `GOOGLE_API_KEY` (무료: https://aistudio.google.com/apikey)
- 또는 `OPENROUTER_API_KEY` (https://openrouter.ai/keys)

**Quickstart 구조**:
```
quickstart/                  # Docker 기반
├── .env.quickstart          # 최소 설정 템플릿
├── sample_data.json         # 25개 FAQ 샘플 데이터
└── load_sample_data.py      # Weaviate 데이터 로드

easy_start/                  # Docker-Free
├── .env.local               # 로컬 설정 템플릿
├── chat.py                  # CLI 챗봇 (Rich UI)
├── load_data.py             # ChromaDB 데이터 로드
└── run.py                   # 원클릭 실행 오케스트레이터
```

**실행 명령어**:
- `make start` - Docker 원클릭 실행 (Weaviate + API + 샘플데이터)
- `make start-down` - 서비스 종료
- `make start-logs` - 로그 확인
- `make easy-start` - Docker-Free 로컬 CLI 챗봇 실행

## 개발 명령어

```bash
# 초기 환경 설정 (spaCy 한국어 모델 포함 자동 설치)
uv sync

# 개발 서버 및 테스트
make dev-reload         # 자동 리로드 (uvicorn --reload)
make test               # 2,200+ 테스트 실행 (외부 로그 차단 격리 환경)
make test-cov           # 테스트 커버리지 리포트

# 코드 품질 관리 (CI/CD 통과 필수)
make lint               # ruff 린트 검사
make type-check         # mypy 엄격 모드 타입 체크
make lint-imports       # 아키텍처 계층 검증 (Import Linter)
```

## 아키텍처 핵심 (v1.0 고도화)

### 1. 지능형 검색 (Hybrid Retrieval)
- **Weaviate**: Dense(의미) + Sparse(BM25) 하이브리드.
- **GraphRAG**: `NetworkXGraphStore`에 벡터 검색 엔진 통합. "SAMSUNG"으로 "삼성전자" 노드 탐색 가능.
- **Reranker v2.1**: 3단계 계층 구조 (approach → provider → model)로 명확한 설정
  - **approach**: `llm`, `cross-encoder`, `late-interaction`, `local` (4종)
  - **provider**: google, openai, jina, cohere, openrouter, sentence-transformers (6종)
  - **v1.2.1 신규**: Cohere (100+ 언어), Local (API 키 불필요), OpenRouter (다양한 LLM 모델 지원)

### 2. 완벽한 보안 (Unified Security)
- **PII Facade**: `PIIProcessor`가 단순 마스킹과 고도화된 AI 리뷰(`PIIReviewProcessor`)를 통합 관리.
- **Admin Auth**: `/api/admin` 하위의 모든 엔드포인트에 `X-API-Key` 인증 전역 적용.

### 3. 운영 유연성 (Dynamic Config)
- **YAML Routing**: `routing_rules_v2.yaml`에서 서비스 핵심 키워드를 관리. 코드 수정 없이 복합 쿼리 판단 로직 변경 가능.
- **환경별 설정**: `app/config/environments/`에 development, test, production 설정 분리. 환경 자동 감지 및 병합.
- **강화된 검증**: Pydantic 기반 설정 검증으로 타입 안전성 및 범위 검증 (temperature, timeout 등).

### 4. 에러 시스템 v2.0 (Bilingual)
- **ErrorCode 기반**: 모든 에러가 구조화된 에러 코드 사용 (예: `GEN-001`, `SEARCH-003`)
- **양언어 자동 전환**: `Accept-Language` 헤더 기반 한국어/영어 메시지 자동 선택
- **사용자 친화적 메시지**: 기술 에러를 해결 방법과 함께 제공
```python
# 새 에러 형식
raise GenerationError(ErrorCode.GENERATION_TIMEOUT, model="claude-sonnet-4-5")
# → 한국어: "AI 모델 응답이 지연되고 있습니다. 해결 방법: 1) 잠시 후 다시 시도..."
# → 영어: "AI model response is delayed. Solutions: 1) Please try again later..."
```

### 5. DI 컨테이너 (Dependency Injection)
- **80+ Provider**: Singleton(70개) + Factory(10개) 패턴 완비
- **9개 명시적 팩토리**: Agent, Evaluator, GraphRAG, Cache, MCP, Ingestion, VectorStore, Retriever, RerankerV2
- **Deprecated 함수 정리 완료**: 모든 전역 헬퍼 함수 제거/리팩토링 완료 (v1.0.7)
  - `get_cost_tracker()`, `get_mongodb_client()`, `get_prompt_manager()` 제거
  - `get_circuit_breaker()` 제거, `get_performance_metrics()` → private 전환
- **테스트 용이성**: 모든 의존성 주입 가능, Mock 교체 용이

### 6. Multi-LLM Factory (v1.0.3)
- **5개 Provider 지원**: Google Gemini, OpenAI GPT, Anthropic Claude, OpenRouter, Ollama
- **자동 Fallback**: 주 LLM 실패 시 설정된 순서대로 자동 전환
- **GPT5QueryExpansionEngine**: `llm_factory` 필수화로 OpenAI 직접 의존성 제거

### 7. Multi Vector DB (v1.0.5)
- **Factory 패턴**: `VectorStoreFactory`, `RetrieverFactory`로 벡터 DB 동적 선택
- **6종 벡터 DB 지원**: 환경변수 `VECTOR_DB_PROVIDER`로 선택
  | Provider | 하이브리드 검색 | 특징 |
  |----------|---------------|------|
  | **weaviate** (기본) | ✅ Dense + BM25 | 셀프호스팅, 하이브리드 내장 |
  | **chroma** | ❌ Dense 전용 | 경량, 로컬 개발용 |
  | **pinecone** | ✅ Dense + Sparse | 서버리스 클라우드 |
  | **qdrant** | ✅ Dense + Full-Text | 고성능 셀프호스팅 |
  | **pgvector** | ❌ Dense 전용 | PostgreSQL 확장 |
  | **mongodb** | ❌ Dense 전용 | Atlas Vector Search |
- **선택적 의존성**: 필요한 DB만 설치 (`uv sync --extra pinecone` 등)

### 8. Observability (v1.0.4)
- **실시간 메트릭**: `/api/admin/realtime-metrics` 엔드포인트
- **캐시 모니터링**: `cache_hit_rate`, `cache_hits`, `cache_misses`, `cache_saved_time_ms`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [notadev-iamaura/OneRAG](https://github.com/notadev-iamaura/OneRAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
