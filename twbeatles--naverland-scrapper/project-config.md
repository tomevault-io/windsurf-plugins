---
trigger: always_on
description: 이 문서는 AI 에이전트가 프로젝트 작업 시 참조할 컨텍스트 정보입니다.
---

# AI Context: Naver Real Estate Crawler Pro Plus (v15.0)

이 문서는 AI 에이전트가 프로젝트 작업 시 참조할 컨텍스트 정보입니다.

## 1. Project Overview
- **Name**: Naver Real Estate Crawler Pro Plus v15.0 (네이버 부동산 매물 크롤러)
- **Goal**: 네이버 부동산 매물 수집, 분석, 모니터링, 관리를 위한 데스크톱 앱
- **Key Features**: 
    - 다중 단지 크롤링 & 그룹 관리
    - Playwright 기본 엔진 + Selenium fallback(`complex` 모드 전용, `geo_sweep`은 Playwright 전용)
    - 지도 탐색 탭 기반 좌표 sweep (APT/VL, 매매/전세/월세)
    - 응답 가로채기 기반 고속 목록 수집 + 필터 통과 매물 우선 모바일 상세 수집
    - 실시간 가격 추세 분석 & 시각화 (히스토그램, 파이차트)
    - Excel/CSV/JSON 내보내기 (템플릿 지원)
    - 카드 뷰/대시보드/즐겨찾기/최근 본 매물
    - 신규/가격 변동/소멸 매물 추적
    - 중개사/연락처/기전세금/갭 분석 필드 저장 및 export
    - **Modern UI - Glassmorphism & Token-based Design (v15.0)**
    - Toast 알림 (페이드 인/아웃 애니메이션)
    - 강력한 에러 핸들링 (RetryHandler, Rate limit 감지)
    - 메모리 관리 (임계치 초과 시 드라이버 재시작)
    - DB 복원 유지보수 모드(복원 중 크롤링/스케줄 차단)
    - SQLite online backup + restore integrity 검증
    - 가격변동 부호 표기 통일(UI/CSV/Excel)
    - 월세 필터 분리(보증금/월세 동시 조건)
    - 원본(raw) 캐시 저장 후 조회 시 재필터링
    - 차단 페이지 감지 시 즉시 실패 처리

## 0. v15.0 Delta Notes (UI/UX Refactoring)
- `styles.py`: 하드코딩된 색상을 `COLORS` 딕셔너리로 추출하고 `_generate_stylesheet` 함수를 도입하여 QSS 기반 동적 테마를 생성합니다.
- `components.py`: 반복적으로 사용되는 '결과 없음' 상태 처리를 위한 `EmptyStateWidget` 컴포넌트가 추가되었습니다. `SummaryCard`와 `SpeedSlider` 등의 인라인 스타일링이 객체 이름 및 QSS 기반으로 리팩토링되었습니다.
- 모든 위젯 및 탭(`CrawlerTab`, `DatabaseTab`, `GroupTab`, `FavoritesTab`, `DashboardWidget` 등)에 `EmptyStateWidget`이 적용되었고, 인라인 QSS가 제거되었습니다.
- 다이얼로그 창(`AboutDialog`, `FilterDialog` 등)의 테마 일관성(HTML 및 색상 포함)이 향상되었습니다.

## 0-1. v14.2 Delta Notes
- `CrawlerTab`:
  - 월세 가격 필터 입력이 `monthly_deposit_*`, `monthly_rent_*`로 확장됨.
  - 고급 필터 UI(버튼/상태 배지)와 결과 파이프라인(테이블/카드뷰) 연동 완료.
  - `retry_on_error=False` 설정 시 크롤러 재시도 횟수는 0으로 강제됨.
- `CrawlerThread`:
  - 캐시 저장 포맷을 `raw_items` 중심으로 전환(legacy `items` 읽기 호환 유지).
  - 차단/방어 페이지 시그널(title/page_source) 감지 시 예외 발생.
  - 스크롤은 내부 컨테이너 우선, 실패 시 window 스크롤 폴백.
  - 월세 필터는 보증금/월세 모두 만족해야 통과.
- `RealEstateApp`:
  - DB 복원 중 유지보수 모드로 전환되어 크롤링/일부 UI 동작이 차단됨.
  - 종료 시 `shutdown_crawl()` 실패하면 앱 종료를 중단하고 DB close를 수행하지 않음.
  - 앱 메뉴의 고급 필터 항목은 CrawlerTab으로 위임됨.
- `NaverURLParser`:
  - 단독 숫자 추출을 라인 단독/명시 문맥으로 제한해 과추출 완화.
- Build:
  - `naverland-scrapper.spec` 점검 결과 본 변경에 대한 추가 hidden import 수정 불필요.

## 0-2. v15.0 Anti-Bot / Geo Expansion (2026-03-06)
- `CrawlerThread`:
  - 엔진 오케스트레이션 레이어로 전환되고 `PlaywrightCrawlerEngine`/`SeleniumCrawlerEngine` 선택 및 fallback을 담당.
  - `crawl_mode=complex|geo_sweep`, `engine_name`, `GeoSweepConfig`, `discovered_complex_signal`을 지원.
- `PlaywrightCrawlerEngine`:
  - 기본 Chromium 엔진, response interception 기반 목록 수집, 필터 통과 매물 우선 모바일 상세 워커 풀 수집을 담당.
  - 지도 sweep에서 발견한 단지를 자동 등록하고 같은 실행 흐름에서 상세 수집까지 연결.
- `ComplexDatabase` / `DataExporter`:
  - `asset_type`, broker/contact, `existing_jeonse_price`, `gap_price`, `gap_ratio` 등 신규 컬럼을 저장/조회/export.
  - `crawl_history`에 `engine`, `mode`, `source_lat/lon/zoom` 메타데이터를 저장.
- UI / Build:
  - `GeoCrawlerTab` 추가, 설정에 `crawl_engine`, `playwright_*`, `geo_*` 항목 추가.
  - `naverland-scrapper.spec`는 Playwright hidden import, runtime hook, Chromium bundle 수집을 포함.

## 2. Technical Stack
- **Language**: Python 3.9+
- **GUI Framework**: `PyQt6` (Widgets, Core, Gui)
- **Web Automation**: 
    - `playwright` (기본 Chromium 엔진, response interception, 필터 통과 매물 우선 모바일 상세 수집)
    - `undetected-chromedriver` (동적 콘텐츠 & 우회)
    - `selenium` (fallback 및 DevTools 계층)
    - `beautifulsoup4` (파싱)
    - `urllib` (API 호출)
    - `winreg` (Windows 레지스트리 기반 Chrome 버전 감지)
- **Data Storage**: 
    - **SQLite** (`data/complexes.db`): 메인 데이터 저장
    - **JSON**: 설정, 프리셋, 캐시, 히스토리
- **Visualization**: `matplotlib` (PyQt6 임베디드)
- **Build Tool**: `PyInstaller`
- **Distribution Profile**: `naverland-scrapper.spec` (기본 `onedir + Chromium bundle`, `NAVERLAND_ONEFILE=1` 시 `onefile`, `NAVERLAND_BUNDLE_CHROMIUM=0` 시 slim)
- **Optional**: `psutil` (메모리 모니터링), `plyer` (알림)

## 3. Architecture (v15.0 Modular)

> **IMPORTANT**: 코드베이스가 모놀리식에서 모듈화 아키텍처로 리팩토링됨

```
src/
├── main.py                     # 진입점
│
├── core/                       # 비즈니스 로직
│   ├── crawler.py              # CrawlerThread (QThread, engine orchestration)
│   ├── database.py             # ConnectionPool, ComplexDatabase
│   ├── item_parser.py          # ItemParser (정규화 파싱)
│   ├── parser.py               # NaverURLParser (RetryHandler 적용)
│   ├── cache.py                # CrawlCache (TTL 기반)
│   ├── analysis.py             # MarketAnalyzer, ComplexComparator
│   ├── export.py               # DataExporter, ExcelTemplate
│   ├── managers.py             # SettingsManager, FilterPresetManager, SearchHistoryManager, RecentlyViewedManager
│   ├── engines/                # 크롤링 엔진 계층
│   │   ├── base.py             # CrawlerEngine 인터페이스
│   │   ├── playwright_engine.py # 기본 Playwright 엔진
│   │   └── selenium_engine.py  # 호환 / fallback 엔진
│   ├── models/                 # 크롤링 DTO / enum
│   │   └── crawl_models.py     # CrawlMode, CrawlRequest, GeoSweepConfig
│   └── services/               # Anti-bot / geo 보조 서비스
│       ├── map_geometry.py     # Mercator 변환, sweep 좌표 생성
│       ├── response_capture.py # 목록 응답 정규화
│       ├── detail_fetcher.py   # 모바일 상세 수집
│       └── gap_analysis.py     # 갭 계산 유틸리티
│
├── ui/                         # 사용자 인터페이스
│   ├── app.py                  # RealEstateApp (QMainWindow)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/twbeatles) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
