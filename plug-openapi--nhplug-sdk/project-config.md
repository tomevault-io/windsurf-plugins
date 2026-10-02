---
trigger: always_on
description: 이 저장소로 개발하는 AI 코딩 에이전트(Antigravity·Cursor·Claude Code 등)는 아래 규칙을 따른다.
---

# AGENTS.md — NH투자증권 Open API 개발 규칙 (AI 에이전트용)

이 저장소로 개발하는 AI 코딩 에이전트(Antigravity·Cursor·Claude Code 등)는 아래 규칙을 따른다.

## 설치 — `pip install nhplug` (패키지명 `nhplug`, 저장소명 `nhplug-sdk`)
- 배포되는 것: `nhplug`(코어) · `nhplug.realtime`(WebSocket) · `nhplug.instruments`(종목마스터 파서 + `.h` 28종).
- 배포되지 않는 것: `snippets/` `examples/` `pipeline/` `guides/` — 저장소를 clone 해서 참고한다.
- `pip install "nhplug[instruments]"` 로 pandas 를 함께 설치하면 마스터가 DataFrame 으로 온다.
- ⚠️ 저장소 경로 `instruments/` 는 설치 시에만 `nhplug/instruments/` 로 매핑된다(`package-dir`).
  **헤더 `.h` 의 GitHub 링크가 포털에 게시돼 있어 저장소 경로를 옮기면 안 된다.**

## 저장소 개요
- `nhplug/` : 공용 클라이언트(토큰 발급·캐시, Input_0 봉투, 헤더 자동) + `realtime.py`. 새 코드는 이걸 재사용한다.
- `snippets/` : 기능 단위 실행 샘플(기능당 폴더 = 호출.py + chk_검증.py). 특정 기능 구현 시 참고.
- `examples/` : 카테고리 통합 예제.
- `pipeline/` : 설계→검증→실행 파이프라인 골격.
- `docs/` : 도메인 명세의 로컬 사본 위치(`scripts/fetch_docs.py` 로 받음, 커밋 안 함). 정본은 도메인 URL(아래).

## 🗺️ 파일 지도 — 무엇을 알고 싶을 때 어디를 여는가

⚠️ **GitHub 은 폴더 목록(`/tree/`)을 크롤러에 막아 둔다.** 아래 링크로 직접 열어야 한다.
파일 목록이 필요하면 이 표와 [README 「구성」 절](https://github.com/PLUG-OpenAPI/nhplug-sdk/blob/main/README.md)을 본다.

| 알고 싶은 것 | 열어볼 파일 |
|---|---|
| REST 호출 · `rsp_cd` 판정 · **연속조회** · **유량 스로틀** | [nhplug/client.py](https://github.com/PLUG-OpenAPI/nhplug-sdk/blob/main/nhplug/client.py) |
| WebSocket 구독 · 포트 라우팅 · **서버 한도** · TLS | [nhplug/realtime.py](https://github.com/PLUG-OpenAPI/nhplug-sdk/blob/main/nhplug/realtime.py) |
| 토큰 발급·캐시 · **호스트 가드** · 브랜드 판정 | [nhplug/auth.py](https://github.com/PLUG-OpenAPI/nhplug-sdk/blob/main/nhplug/auth.py) |
| 오류 분류(`config`·`auth`·`rate_limit`·`network`·`http`) | [nhplug/errors.py](https://github.com/PLUG-OpenAPI/nhplug-sdk/blob/main/nhplug/errors.py) |
| `.env` 탐색 순서 | [nhplug/\_env.py](https://github.com/PLUG-OpenAPI/nhplug-sdk/blob/main/nhplug/_env.py) |
| **실시간 채널 27종** · `tr_key` 대응표 | [docs/realtime_channels.md](https://github.com/PLUG-OpenAPI/nhplug-sdk/blob/main/docs/realtime_channels.md) |
| **종목마스터 28종** 목록 · 파서 사용법 | [instruments/README.md](https://github.com/PLUG-OpenAPI/nhplug-sdk/blob/main/instruments/README.md) |
| `.mst` 파싱 구현 | [instruments/master.py](https://github.com/PLUG-OpenAPI/nhplug-sdk/blob/main/instruments/master.py) |
| **SDK 없이** 원시 호출(urllib·requests) | [snippets/standalone/README.md](https://github.com/PLUG-OpenAPI/nhplug-sdk/blob/main/snippets/standalone/README.md) |
| 계좌구분(`acct_type`) 판정 예제 | [snippets/common/list_accounts](https://github.com/PLUG-OpenAPI/nhplug-sdk/blob/main/snippets/common/list_accounts/list_accounts.py) |
| 실시간 구독 예제·검증 | [snippets/krstock/realtime_execution](https://github.com/PLUG-OpenAPI/nhplug-sdk/blob/main/snippets/krstock/realtime_execution/chk_realtime_execution.py) |
| AI IDE 규칙 파일(고객 배포용) | [templates/README.md](https://github.com/PLUG-OpenAPI/nhplug-sdk/blob/main/templates/README.md) |
| 설치부터 첫 실행까지 절차 | [guides/antigravity.md](https://github.com/PLUG-OpenAPI/nhplug-sdk/blob/main/guides/antigravity.md) |
| **대화로 조회**(MCP 서버) | [nhplug-mcp](https://github.com/PLUG-OpenAPI/nhplug-mcp) |

**명세 자체**를 알고 싶으면 저장소가 아니라 도메인을 본다 → 아래 「문서 (Source of Truth)」 절.

## 문서 (Source of Truth) — 도메인이 정본(SSOT)
- 전체 개요·인증·공통 규약: https://www.nhplug.com/llms.txt (나무) · https://www.n2plug.com/llms.txt (N2)
- 자산군 정본(openapi.json·overview.md·README.md): https://www.nhplug.com/openapi-docs/<domain>/ (나무) · https://www.n2plug.com/openapi-docs/<domain>/ (N2)
  (domain: common · krstock · gbstock · krfuture · gbfuture · krbond · krgold)
- 엔드포인트·필드·형식은 위 **도메인 openapi.json** 을 정본으로 따른다. 로컬 사본이 필요하면 `python scripts/fetch_docs.py` 로 `docs/` 에 받는다(커밋 안 함).
- 🔴 **에러 처리(가장 중요)**: **SDK 는 업무 성공/실패를 판정하지 않는다.** 기준은 HTTP 상태코드 하나다.

  | HTTP | `call()` 동작 |
  |---|---|
  | **200** | 응답 본문을 **그대로** 반환. 예외 없음. `rsp_cd`·`rsp_msg` 도 손대지 않음 |
  | **200 아님** | `NhplugError`(category: `config`\|`auth`\|`rate_limit`\|`network`\|`http`) — 본문은 `.raw` 에 원문 그대로 |

  ```python
  from nhplug import call, status_of

  data = call("/krstock/quote/v1/currentPrice", {"iem_cd": "005930", "market_cd": "UNT"})
  rsp_cd, rsp_msg = status_of(data)      # 꺼내 줄 뿐 — 판정하지 않는다
  print(rsp_msg)                         # 서버가 보낸 문장 그대로

  price = (data.get("Output_0") or {}).get("stck_prpr")
  if price is None:                      # 기대한 값이 왔는지는 호출자가 직접 확인
      print("⚠️ 현재가 없음. 위 rsp_msg 확인")
  ```

  **왜 판정하지 않나**: **같은 `rsp_cd` 값이 API 에 따라 정상일 수도 오류일 수도 있다.** 어떤 코드 목록도
  전수가 될 수 없고, 판정하면 정상 응답을 실패로(또는 그 반대로) 오판해 사용자에게 잘못 알린다.

  ⚠️ **`if rsp_cd == "00000"` 같은 코드를 절대 쓰지 말 것.** 코드값을 비교하는 판정은 API 를 바꾸면 무너진다.
  **`rsp_msg` 문장을 읽고 판단한다.** 규약 정본은 [llms.txt](https://www.nhplug.com/llms.txt) (N2: https://www.n2plug.com/llms.txt) 의 「성공 판정」 절.

  🔴 **주문 등 되돌릴 수 없는 처리 전에는 직전 응답의 `rsp_msg` 를 반드시 확인한다.** SDK 가 막아 주지 않는다.

  ⚠️ 0.4.0 에서 `is_success()`·`success_codes()`·`NHPLUG_SUCCESS_CODES`·`category="business"` 가 **삭제됐다.**
  코드값 목록(`00000`·`00166`·`00221`·`13578`)도 코드베이스에서 사라졌다 — **되살리지 말 것.**
- **토큰(중요)**: 24시간 유효. `~/.nhplug/` 에 **파일 캐시**되어 프로세스가 바뀌어도 재사용된다(재발급 1회 = 보안 알림 1건).
  캐시 파일 권한은 OS 기본값이다(별도 chmod 없음). 공유 환경이면 `NHPLUG_TOKEN_CACHE_DIR` 로 옮기거나 `NHPLUG_TOKEN_CACHE=0`.
  **재발급은 401(토큰 무효)일 때만.** `429`(유량 초과) 재시도에는 기존 토큰을 그대로 쓴다 — 토큰을 직접 재발급하는 코드를 새로 만들지 말 것.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PLUG-OpenAPI/nhplug-sdk](https://github.com/PLUG-OpenAPI/nhplug-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
