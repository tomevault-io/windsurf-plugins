---
trigger: always_on
description: 이 문서는 Claude Code에서 이 레포지토리 작업 시 참조하는 개발자 가이드입니다.
---

# Developer Guide - Vault Intelligence System V2

이 문서는 Claude Code에서 이 레포지토리 작업 시 참조하는 개발자 가이드입니다.

사용자의 요청을 처리하기 위해 검색이 필요한 경우 vault intelligence 시스템을 이용해서 사용자의 vault 문서를 효과적으로 검색해서 사용자의 요구사항에 대응해 줘

## 🔍 CLI 빠른 참조 (Quick Reference)

**⚠️ 중요: 아래 옵션을 정확히 사용하세요. 자주 실수하는 옵션들을 주의하세요!**

> `vis`는 메인 명령어이며, 하위 호환을 위해 `vault-intel`도 사용 가능합니다.

### 기본 검색
```bash
# 기본 하이브리드 검색
vis search "TDD"

# 검색 방법 지정
vis search "TDD" --search-method semantic   # 의미적 검색
vis search "TDD" --search-method keyword    # 키워드 검색
vis search "TDD" --search-method hybrid     # 하이브리드 (기본값)
vis search "TDD" --search-method colbert    # ColBERT 토큰 검색
```

### 고급 검색 옵션
```bash
# 재순위화 (정확도 향상)
vis search "TDD" --rerank

# 쿼리 확장 (포괄성 향상)
vis search "TDD" --expand                    # 동의어 + HyDE
vis search "TDD" --expand --no-hyde         # 동의어만
vis search "TDD" --expand --no-synonyms     # HyDE만

# 최고 품질 검색 (모든 기능 결합)
vis search "TDD" --rerank --expand

# 결과 수 및 임계값 조정
vis search "TDD" --top-k 20 --threshold 0.5

# 중심성 점수 반영
vis search "TDD" --with-centrality

# 결과 파일 저장
vis search "TDD" --output results.md
vis search "TDD" --output                    # 기본 파일명으로 저장

# 데이터 디렉토리 지정 (기본값: ~/git/vault-intelligence)
vis search "TDD" --data-dir /custom/path
```

### 기타 주요 명령어
```bash
# 관련 문서 찾기
vis related "문서명.md" --top-k 10

# 지식 공백 분석
vis analyze-gaps --top-k 20

# 주제별 문서 수집
vis collect "TDD" --output collection.md

# MOC 생성
vis generate-moc "TDD" --top-k 50

# 자동 태깅
vis tag "문서명.md"
vis tag "폴더명/" --recursive

# 인덱싱
vis reindex                    # 기본 재인덱싱
vis reindex --with-colbert     # ColBERT 포함
vis reindex --force            # 강제 전체 재인덱싱

# 태그 분석
vis list-tags

# 주제별 문서 연결
vis connect-topic "TDD" --dry-run    # 미리보기
vis connect-topic "TDD"              # 실행

# 연결 상태 확인
vis connect-status

# 고립 태그 정리
vis clean-tags --dry-run
vis clean-tags
```

### 데몬 관리 (visd)
```bash
# vis search는 visd 데몬이 필수
visd start                             # 백그라운드 시작
visd status                            # 상태 확인 (PID, 문서 수, 인덱스)
visd stop                              # 중지
visd restart                           # 재시작
visd logs 30                           # 최근 로그 30줄

# HTTP API 직접 호출
curl http://localhost:8741/health
curl -s --get --data-urlencode "query=TDD" "http://localhost:8741/search?rerank=true&top_k=10"
curl -X POST "http://localhost:8741/reindex?force=true"
```

### ⚠️ 자주 실수하는 옵션들
```bash
# ❌ 잘못된 사용
vis search "TDD" --method semantic        # --method (X)
vis search "TDD" --k 20                   # --k (X)
vis search "TDD" --top 20                 # --top (X)
vis search "TDD" --output-file out.md     # --output-file (X)
vis search "TDD" --reranking              # --reranking (X)
vis search --query "TDD"                  # --query (X) → positional로 변경됨

# ✅ 올바른 사용
vis search "TDD" --search-method semantic # --search-method (O)
vis search "TDD" --top-k 20               # --top-k (O)
vis search "TDD" --output out.md          # --output (O)
vis search "TDD" --rerank                 # --rerank (O)
```

### 검색 방법 선택 가이드
| 검색 방법 | 사용 상황 | 속도 | 정확도 |
|-----------|-----------|------|--------|
| `semantic` | 개념적, 의미적 검색 | ⚡⚡⚡ | ⭐⭐⭐ |
| `keyword` | 정확한 용어 검색 | ⚡⚡⚡ | ⭐⭐ |
| `hybrid` | 일반적인 모든 검색 (권장) | ⚡⚡⚡ | ⭐⭐⭐⭐ |
| `colbert` | 긴 문장, 복합 개념 | ⚡⚡ | ⭐⭐⭐⭐ |
| `--rerank` | 고정확도 필요 시 | ⚡⚡ | ⭐⭐⭐⭐⭐ |
| `--expand` | 포괄적 검색 필요 시 | ⚡ | ⭐⭐⭐⭐ |

## 📖 문서 구조

- **[README.md](README.md)** - 프로젝트 개요 및 빠른 시작
- **[사용자 가이드](docs/USER_GUIDE.md)** - 완전한 사용법 매뉴얼
- **[실전 예제](docs/EXAMPLES.md)** - 다양한 활용 사례
- **[문제 해결](docs/TROUBLESHOOTING.md)** - 기술 지원 가이드
- **이 문서** - 개발자 및 기여자를 위한 상세 가이드

## 🏗️ 시스템 아키텍처

### 핵심 모듈 구조

```
src/
├── constants.py                    # 공유 상수 (DEFAULT_PORT, PID_FILE)
├── server.py                       # FastAPI 데몬 서버 (HTTP API)
├── server_runner.py                # 서버 프로세스 런처
├── client.py                       # HTTP 클라이언트 (visd용)
├── core/                           # 핵심 엔진
│   ├── sentence_transformer_engine.py  # BGE-M3 임베딩 엔진
│   ├── embedding_cache.py              # SQLite 캐싱 시스템
│   └── vault_processor.py              # Vault 파일 처리
├── features/                       # 기능 모듈
│   ├── advanced_search.py              # 다층 검색 엔진
│   ├── reranker.py                     # Cross-encoder 재순위화
│   ├── colbert_search.py               # ColBERT 토큰 검색
│   ├── query_expansion.py              # 쿼리 확장 (동의어 + HyDE)
│   ├── semantic_tagger.py              # 자동 태깅 시스템
│   ├── content_clusterer.py            # 문서 클러스터링 (Phase 9)
│   ├── document_summarizer.py          # 문서 요약 시스템 (Phase 9)
│   ├── learning_reviewer.py            # 학습 리뷰 시스템 (Phase 9)
│   ├── knowledge_graph.py              # 지식 그래프 분석
│   ├── moc_generator.py                # MOC 자동 생성
│   ├── topic_collector.py              # 주제별 문서 수집
│   └── duplicate_detector.py           # 중복 문서 감지
├── utils/                          # 유틸리티
│   └── claude_code_integration.py      # Claude Code LLM 통합
└── __main__.py                     # CLI 엔트리 포인트
```

### 데이터 계층

- **SQLite 캐시**: `cache/embeddings.db` - 임베딩 벡터 영구 저장
- **메타데이터**: `cache/metadata.json` - 문서 메타데이터 캐싱
- **설정**: `config/settings.yaml` - 시스템 전역 설정
- **모델**: `models/` - 다운로드된 BGE-M3 모델

## 🚀 개발 환경 설정

### 1. 설치

```bash
# 방법 A: pipx 설치 (권장 - 어디서든 vis/vault-intel 명령어 사용 가능)
pipx install -e ~/git/vault-intelligence

# 방법 B: 개발 의존성 직접 설치

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [msbaek/vault-intelligence](https://github.com/msbaek/vault-intelligence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
