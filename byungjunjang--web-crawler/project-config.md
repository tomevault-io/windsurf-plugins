---
trigger: always_on
description: 이 프로젝트는 사용자가 URL과 수집 항목을 자연어로 설명하면, 자동으로 해당 웹사이트를 정찰하고 데이터를 대량 수집하여 엑셀 파일로 정리해주는 에이전트입니다.
---

# 범용 웹 크롤링 에이전트

## 프로젝트 개요

이 프로젝트는 사용자가 URL과 수집 항목을 자연어로 설명하면, 자동으로 해당 웹사이트를 정찰하고 데이터를 대량 수집하여 엑셀 파일로 정리해주는 에이전트입니다.

---

## 최초 환경 셋업 (클론 직후 1회, 수집 전 확인)

수집 시도 전에 환경을 준비한다. **한 명령**으로 단계별 설치+검증(이미 된 단계는 skip):

```powershell
powershell -ExecutionPolicy Bypass -File scripts\setup.ps1     # Windows (venv 자동 생성)
```
```bash
python -m venv .venv && . .venv/bin/activate && python scripts/bootstrap.py   # macOS/Linux
```

수동/디버깅 시 실제 동작 명령:
```bash
pip install -r requirements.txt          # scrapling[fetchers] 포함 — fetcher 런타임 일괄
scrapling install                        # Chromium 1회 (내부에서 playwright install chromium 수행 — 따로 또 X)
npm.cmd install -g agent-browser ; agent-browser.cmd install   # 표준 정찰 도구 (PowerShell은 .cmd)
python scripts/preflight.py              # 검증: core / agent-browser 분리 PASS·WARN·FAIL
```

- `python -m scrapling`은 동작 안 함 → `scrapling install`(venv 활성화) 또는 `.\.venv\Scripts\scrapling.exe install`.
- 검증은 `scripts/preflight.py`(설치 안 함). core 통과·agent-browser 실패면 "전체 설치 미완료"(exit 1).
- 전체 가이드(비개발자용 포함)는 `README.md`의 "처음 설치하기" 참조.

---

## ★ 절대 규칙 0: 도메인 히스토리 우선 (모든 수집의 시작)

새 수집 요청을 받으면 **정찰하기 전에 반드시** 이 두 가지를 먼저 본다:

1. **`fingerprints/<sanitized_domain>/profile.json`** — 그 도메인의 검증된 수집 레시피 (fetcher_type, antibot_type/strategy, selectors, api_endpoints, pagination, notes)
2. **`output/<도메인>/`** — 그 도메인에서 이전에 실행된 수집 작업 폴더들 (그 안의 `crawl_script.py`/`raw_data.json`은 profile.json에 안 박힌 미세 디테일의 보조 reference)

### 운영 흐름

```
[수집 요청] → 도메인 추출 → fingerprints/<sanitized_domain>/profile.json 조회
                                    │
                ┌───────────────────┴───────────────────┐
                │ 프로필 있음                            │ 프로필 없음
                ▼                                       ▼
       ┌────────────────────────┐         ┌─────────────────────────┐
       │ 1. notes 먼저 읽고     │         │ 정찰 (Step 2) 부터 시작 │
       │    전략에 반영          │         │ — 모든 단계 풀 실행      │
       │ 2. fetcher_type/       │         └─────────────────────────┘
       │    antibot_strategy     │
       │    그대로 채택          │
       │ 3. selectors/endpoints │
       │    재사용 시도          │
       │ 4. last_used가 3개월+  │
       │    오래됐거나 사용자가  │
       │    "최신 구조로"를      │
       │    명시했으면 정찰 추가  │
       │ 5. 검증 실패하면 셀렉터/│
       │    엔드포인트만 정찰    │
       └────────────────────────┘
                │
                ▼
       채택한 값이 사다리 B(4단 이상)인가?
                │
        ┌───────┴────────┐
        │ 아니오(1~3단)   │ 예 — consent 기록이 있나?
        ▼                ├── 있음 → 통지 없이 진행 (sticky)
   그대로 진행            └── 없음 → **이번이 최초 통과다. 그대로 통지한다**
        │                          (프로필 재사용은 이음매 면제가 아니다)
        ▼
       정찰 스킵하고 Step 3 (수집 전략 수립)으로 점프
```

> **프로필이 있다는 사실 자체는 게이트를 면제하지 않는다.** 면제하는 것은 `consent` 기록이다.
> `antibot_strategy` 가 4단 이상인데 `consent` 가 없는 프로필은 "이 사용자는 아직 통지받은 적이
> 없다" 는 뜻이다 — 그대로 실행하면 통지 없이 사다리 B 를 돌게 되고, 수집이 다 끝난 뒤
> `save()` 가 `ConsentRequired` 로 막아 있지도 않았던 통지를 기록하도록 떠밀린다.
>
> **그래서 통지는 도메인당 1회가 아니라 이음매를 통과할 때마다 1회다.** 사다리 A 로 내려간
> 수집에서 프로필이 배포 대상이 되면 `save()` 가 `consent` 를 지운다 — 사용자의 통지 이력을
> 배포되는 프로필에 담아 내보낼 수 없기 때문이다. 그 도메인이 나중에 새 보호를 걸면 들고 있는
> 기록이 없으므로 다시 통지한다. **사이트가 새로 막은 것은 달라진 상황이니 그게 맞다.**

### 절대 안 되는 것

- profile.json이 있는데 그걸 무시하고 정찰부터 다시 하기 — 5~20분의 비싼 작업을 매번 반복하는 행위
- profile.json의 `notes` 필드를 읽지 않고 전략 세우기 — notes에는 LLM이 자동으로 못 알아내는 결정적 메타 정보가 박혀 있다 ("리스트는 SSR HTML, 상세는 XHR JSON — 2단으로 충분", "review API는 JSON 아니라 HTML 반환" 등)
- 수집 성공 후 profile.json 저장/갱신을 빠뜨리기 — Step 5-A 필수 게이트. 누락 시 "파이프라인 미완료" 보고

<!-- BEGIN GENERATED: domain-list -->
<!-- 이 블록은 scripts/sync_domain_list.py 가 생성한다. 직접 수정하지 말 것. -->

### 알려진 도메인 (14개 profile commit됨)

`books.toscrape.com`, `builtini.co.kr`, `celimax.co.kr`, `data.seoul.go.kr`, `db.itkc.or.kr`, `g2b.go.kr`, `guesskorea.com`, `made-in-china.com`, `wanted.co.kr`, `www.11st.co.kr`, `www.fss.or.kr`, `www.gsmarena.com`, `www.k-startup.go.kr`, `www.kurly.com` — 이 도메인들은 정찰 없이 바로 수집 시도 가능.

<!-- END GENERATED: domain-list -->

### profile 조회/저장 코드

```python
from domain_profile import DomainProfile
profile_mgr = DomainProfile()  # base_dir=./fingerprints

# Step 1-A: 조회
if profile_mgr.exists(domain):
    profile = profile_mgr.load(domain)
    # notes/fetcher_type/antibot_strategy/api_endpoints/selectors 활용

# Step 5-A: 수집 성공 후 저장 (필수 게이트)
profile_mgr.save(domain, {
    "domain": domain,
    "capability": "<static|js_render|api|session>",   # ★ SSOT — 능력 수준. 비워 두면 save() 가 fetcher_type 에서 채운다
    "fetcher_type": "<yt-dlp|RSS|oEmbed|Jina|Fetcher|FetcherSession|DynamicFetcher|DynamicSession|Spider|playwright_spa_intercept|curl_cffi_grid|StealthyFetcher|chrome_cdp|API_SESSION>",   # 파생 — 현재 엔진에서의 구현체. 앞 4개는 Phase 0 공인 우회로
    "antibot_type": "<none|cloudflare|akamai|spa_session|naver_antibot|other>",
    "antibot_strategy": "<none|playwright_intercept|impersonate|curl_cffi_grid|stealthy|chrome_cdp|naver_antibot|authenticated_browser>",   # 실제로 쓴 대응. 사다리 B 를 썼으면 반드시 그 값을 적는다
    "site_type": "<static|csr|api|spa_session|akamai>",
    "selectors": {...},
    "pagination": {...},
    "api_endpoints": [...],
    "notes": "<다음 사람이 정찰 없이 바로 수집할 수 있는 결정적 한두 줄>",
    # 사다리 B(4단 이상)로 수집했을 때만. **실제로 통지했고 사용자가 '진행' 을 고른 경우에만 적는다** —
    # 그 일이 없었으면 이 블록을 적지 않는다. 근거가 아니라 선택을 적는다.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [byungjunjang/web-crawler](https://github.com/byungjunjang/web-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
