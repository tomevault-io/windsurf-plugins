---
trigger: always_on
description: > Claude Opus 4.6가 프로젝트를 이해하고 작업을 계속할 수 있도록 작성된 컨텍스트 파일입니다.
---

# 🤖 CLAUDE.md - AI 에이전트 프로젝트 컨텍스트

> Claude Opus 4.6가 프로젝트를 이해하고 작업을 계속할 수 있도록 작성된 컨텍스트 파일입니다.
> **새 대화 시작 시 `@CLAUDE.md`를 참조하세요.**

---

## 📋 프로젝트 개요

| 항목 | 값 |
|------|-----|
| **프로젝트명** | KakaoTalk Chat Summary |
| **목적** | 카카오톡 대화를 LLM으로 요약하고 관리하는 데스크톱 앱 |
| **언어** | Python 3.11+ |
| **GUI** | PySide6 (Qt for Python) |
| **DB** | SQLite + SQLAlchemy ORM |
| **버전** | v2.9.3 |
| **최종 업데이트** | 2026-04-24 |

---

## 🏗️ 프로젝트 구조

```
kakao-chat-summary/
├── src/
│   ├── app.py                 # 앱 진입점 (QApplication)
│   ├── ui/
│   │   ├── __init__.py
│   │   ├── main_window.py     # 메인 GUI (5000+ lines)
│   │   └── styles.py          # 카카오톡 스타일 테마
│   ├── db/
│   │   ├── __init__.py        # get_db() export
│   │   ├── database.py        # Database 클래스
│   │   └── models.py          # SQLAlchemy 모델 5개
│   ├── file_storage.py        # FileStorage 클래스
│   ├── full_config.py         # Config 클래스 (LLM 설정)
│   ├── parser.py              # KakaoLogParser 클래스
│   ├── detail_prompt.py       # 상세 분석 프롬프트 + HTML 템플릿 + LLM API
│   ├── url_extractor.py       # URL 추출 (마크다운 + HTML 파싱)
│   ├── import_to_db.py        # DB import 유틸
│   └── scheduler/
│       ├── __init__.py
│       └── tasks.py           # SyncScheduler (프레임워크 구현, 메인 앱 미연동)
├── data/
│   ├── db/                    # SQLite 데이터베이스
│   │   └── chat_history.db
│   ├── original/              # 원본 대화 (일별)
│   │   └── <채팅방>/
│   │       └── <채팅방>_YYYYMMDD_full.md
│   ├── summary/               # LLM 요약 (일별)
│   │   └── <채팅방>/
│   │       └── <채팅방>_YYYYMMDD_summary.md
│   ├── url/                   # URL 목록 (채팅방별 3개 파일)
│   │   └── <채팅방>/
│   │       ├── <채팅방>_urls_recent.md
│   │       ├── <채팅방>_urls_weekly.md
│   │       └── <채팅방>_urls_all.md
│   └── detail_summary/        # 상세 분석 HTML (v2.8.0)
│       └── <채팅방>/
│           └── <채팅방>_YYYYMMDD_detail.html
├── output/                    # CLI 스크립트 (src/manual/) 출력 디렉터리
├── upload/                    # 파일 업로드 기본 디렉터리
├── logs/                      # 로그 (summarizer_YYYYMMDD.log)
├── docs/                      # 문서 (01-prd ~ 06-tasks)
├── .env.local                 # API 키 (gitignore)
├── env.local.example          # API 키 예제
├── requirements.txt
├── .gitignore
├── README.md
└── CLAUDE.md                  # 이 파일
```

---

## 🗃️ 데이터베이스 스키마 (5개 테이블)

### ChatRoom
```python
class ChatRoom(Base):
    __tablename__ = 'chat_rooms'
    id = Column(Integer, primary_key=True)
    name = Column(String(255), nullable=False)
    file_path = Column(String(512))
    participant_count = Column(Integer, default=0)
    last_sync_at = Column(DateTime)
    created_at = Column(DateTime, default=datetime.now)
    # Relationships: messages, summaries, sync_logs, urls
```

### Message
```python
class Message(Base):
    __tablename__ = 'messages'
    id = Column(Integer, primary_key=True)
    room_id = Column(Integer, ForeignKey('chat_rooms.id'))
    sender = Column(String(255), nullable=False)
    content = Column(Text)
    message_date = Column(Date, nullable=False)
    message_time = Column(Time)
    raw_line = Column(Text)
    created_at = Column(DateTime)
    # UniqueConstraint: (room_id, sender, message_date, message_time, content)
```

### Summary
```python
class Summary(Base):
    __tablename__ = 'summaries'
    id = Column(Integer, primary_key=True)
    room_id = Column(Integer, ForeignKey('chat_rooms.id'))
    summary_date = Column(Date, nullable=False)
    summary_type = Column(String(50))  # 'daily', '2days', 'weekly'
    content = Column(Text)
    llm_provider = Column(String(100))
    token_count = Column(Integer)
    created_at = Column(DateTime)
```

### SyncLog
```python
class SyncLog(Base):
    __tablename__ = 'sync_logs'
    id = Column(Integer, primary_key=True)
    room_id = Column(Integer, ForeignKey('chat_rooms.id'))
    status = Column(String(50))  # 'success', 'failed', 'partial'
    message_count = Column(Integer)
    new_message_count = Column(Integer)
    error_message = Column(Text)
    synced_at = Column(DateTime)
```

### URL
```python
class URL(Base):
    __tablename__ = 'urls'
    id = Column(Integer, primary_key=True)
    room_id = Column(Integer, ForeignKey('chat_rooms.id'))
    url = Column(Text, nullable=False)
    descriptions = Column(Text)  # " / " 구분자
    source_date = Column(Date)
    created_at = Column(DateTime)
    updated_at = Column(DateTime)
    # UniqueConstraint: (room_id, url)
```

---

## 🖥️ GUI 구조 (main_window.py)

### 메인 윈도우 레이아웃
```
┌─────────────────────────────────────────────────────────┐
│ 메뉴바                                                   │
│ ├─ 파일: 채팅방 추가, 채팅방 삭제, 종료                   │
│ ├─ 도구: 상세 분석 생성, 전체 채팅방 상세분석/URL,          │
│ │       백업/복원, DB 복구, 채팅방 복구, 설정               │
│ └─ 도움말: 정보                                          │
├──────────────┬──────────────────────────────────────────┤
│              │  QTabWidget (4개 탭)                      │
│  채팅방 목록  │  ├─ 📊 대시보드                           │
│  (QListWidget)│  ├─ 📅 날짜별 상세 분석                    │
│              │  ├─ 🔗 URL 정보 (동기화/복구 버튼 포함)    │
│              │  └─ 🔧 기타 (통계 갱신 등)                │
│              │                                          │
│ [➕ 채팅방]   │                                          │
│ [📤 업로드]  │                                          │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YeonHongMin/kakao-chat-summary](https://github.com/YeonHongMin/kakao-chat-summary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
