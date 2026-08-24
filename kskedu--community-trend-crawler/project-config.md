---
trigger: always_on
description: 커뮤니티 사이트 인기글을 수집해 Supabase에 저장하는 크롤러.
---

# community-trend-crawler

커뮤니티 사이트 인기글을 수집해 Supabase에 저장하는 크롤러.
GitHub Actions로 주기적 실행.

## 참조 워크플로우

이 프로젝트는 StartHub 서비스의 커뮤니티/키워드/뉴스 크롤러로 운영된다.
작업 승인, 검증, diff review-only, allowed files stage, commit/push 기준은 StartHub의 워크플로우 문서를 따른다.

- [../StartHub/docs/workflow/ai-workflow.md](../StartHub/docs/workflow/ai-workflow.md)
- [../StartHub/docs/workflow/ai-workflow-git.md](../StartHub/docs/workflow/ai-workflow-git.md)

단, 운영 DB 마이그레이션, 운영 데이터 삭제/복구, secret/env 변경, 외부 서비스 설정 변경은 별도 사용자 확인 후 진행한다.

## 트리거 방식 (2026-05-05~, 2026-07 단일화)

정기 실행원은 **cron-job.org → GitHub `workflow_dispatch` API** 하나로 **단일화**한다.
GitHub Actions 무료 정시 cron 큐 지연(최대 3시간+) 회피가 도입 배경이다.

**2026-07 변경**: crawl.yml 의 GitHub 자체 `on.schedule`(`17`/`47`)을 **제거**했다.
cron-job.org 와 동일 :17/:47·동일 모드로 **이중 트리거**되어 news_top 을 연속 저장
(last-write-wins)·movement 왜곡·큐 적체를 유발했기 때문이다(실운영 재현). 이제
정기 실행은 cron-job.org 의 workflow_dispatch 만이 권위 실행원이다.

- **권위 정기 실행원 = cron-job.org**: 매시 KST HTTP POST → workflow_dispatch
  - **호출 endpoint**: `POST https://api.github.com/repos/kskedu/community-trend-crawler/actions/workflows/crawl.yml/dispatches`
  - **Headers**: `Authorization: Bearer <PAT>` + `Accept: application/vnd.github.v3+json` + `Content-Type: application/json`
  - **PAT**: classic, scopes: `repo` (private 레포)
  - **Body 계약(중요)**: **`:17` 호출 → `{"ref":"main","inputs":{"mode":"full"}}` / `:47` 호출 → `{"ref":"main","inputs":{"mode":"news_top_only"}}`**.
    - crawl.yml 은 `inputs.mode` 로만 모드를 판정한다(schedule 제거로 `github.event.schedule` 분기 소멸). mode 를 생략(`{"ref":"main"}`)하면 crawl.yml 이 안전하게 `full` 로 정규화하지만, 그러면 :47 의 news_top_only 갱신 cadence 가 사라지므로 **두 job 이 각각 올바른 mode 를 전달해야 한다.**
    - ⚠️ cron-job.org 실제 설정은 이 저장소로 확인할 수 없다 → 아래 "외부 확인 게이트" 참조.
- **보조 실행원 = healthcheck.yml (stale 복구 전용)**: 매시 :23, `community_posts` stale(≥120m)/down 시에만 crawl.yml 을 workflow_dispatch. **dispatch 전 crawl.yml 의 queued/in_progress run 을 확인**해 이미 실행 중이면 생략한다(중복 완화, check-then-act·비원자적). run 조회 실패 시 **fail-closed**(dispatch 생략). 텔레그램 알림 유지.
- **수동 실행 = 비상·검증 전용**: crawl.yml workflow_dispatch(UI 또는 `gh workflow run`, mode 선택, default full).

### 장애 시 복구 절차
1. GitHub Actions 상태(githubstatus.com Actions)가 partial_outage/major_outage 면 **두 workflow 를 Disable**(`gh workflow disable crawl.yml healthcheck.yml`)해 큐 적체·유입을 멈춘다.
2. 정체된 queued/in_progress run 을 확인(`gh run list --status queued/in_progress`)하고 필요 시 취소.
3. Actions 가 Operational 로 복귀하고 queued/in_progress 0 인지 확인.
4. cron-job.org 설정(아래 게이트)을 확인한 뒤 crawl.yml 만 Enable → 다음 cron-job.org dispatch 1건 자연 관찰 → 정상 확인 후 healthcheck.yml Enable.

### 외부 확인 게이트 (merge·Enable 전 필수)
cron-job.org 설정은 코드로 검증 불가. 재활성화 전 **사용자가 cron-job.org 콘솔에서** 확인:
- `:17` job 이 `inputs.mode=full` 을 전달하는지
- `:47` job 이 `inputs.mode=news_top_only` 를 전달하는지
- 대상 `ref=main` 인지
- 동일 시각 중복 등록된 cron job 이 없는지
- 인증 PAT 가 유효한지

### 잔여 P2 후속 과제 (이번 단일화로 미해소)
1. **healthcheck 중복 확인의 비원자성**: check-then-act 라 두 호출 동시 관측 시 이중 dispatch 가능(정상 케이스 완화일 뿐, 0건 보장 아님).
2. **full ↔ news_top_only shared news_top 경합 — 앱단 freshness guard 로 완화 완료 (2026-07-20, `e7240e3`)**: 아래 "news_top freshness guard" 절 참조. **완전 원자화는 미해소**(TOCTOU 잔존, 항목 3 참조).
3. **DB optimistic guard / 실행 세대 보호 — 근본(원자) 해결은 미착수**: 현재는 앱단에서 "실행 시작 시점에 읽은 previous" 와 비교하는 **best-effort** 뿐이라, read~write 사이 다른 run 이 더 최신을 write 하는 순수 TOCTOU 는 못 막는다. 근본 해결은 upsert 를 **DB RPC 조건부 write**(`generated_at` compare-and-set)로 바꾸는 것 — service_role/migration 동반하는 별도 PR. concurrency(모드별 group, `cancel-in-progress:false`)는 **healthcheck 복구 dispatch 의 pending 취소(유실) 위험** 때문에 채택하지 않기로 결정함(2026-07-20 계획 리뷰, GitHub Actions 표준 concurrency 는 새 pending 이 기존 pending 을 취소).

### news_top freshness guard (2026-07-20, `e7240e3`)
`run_news_briefing()` 이 upsert 직전 새 payload 의 `generated_at` 을 직전에 읽은 `previous.generated_at` 과 비교해, **새 값이 previous 보다 명확히 과거일 때만** upsert 를 건너뛴다(오래된 실행이 최신 news_top 을 덮어쓰는 것 방지). 동일 시각·최신·결측·비문자열·파싱 실패는 전부 write 허용(fail-open) — 정상 신선 실행을 막지 않는다.
- helper: `main.py` `_parse_generated_at` / `_is_stale_news_top_write` (순수 함수, 테스트 가능).
- stale 시: news_top upsert 만 생략. `keyword_cache`·`community_posts` 경로는 영향 없음. 후보/decisions/rejection_counts 는 그대로 진단 DB 에 보존되고, `status=skipped` / `skip_reason=STALE_WRITE_SKIPPED`(`news/diagnostics.py` `SKIP_REASON_STALE_WRITE`) 로 기록돼 **발행 성공 집계에서 제외**된다.
- 근거는 `thresholds` JSONB 의 `stale_write_v1` namespace 에 저장(`previous_generated_at`/`new_generated_at`/`mode`/`run_id`/`comparison`/`result`, secret·기사 본문 없음).
- **선행조건**: StartHub `news_keyword_runs.skip_reason` CHECK 에 `STALE_WRITE_SKIPPED` 가 등록돼 있어야 한다(StartHub PR #73, migration `docs/migrations/supabase-news-diag-skip-reason-stale-*.sql`, 운영 적용·postcheck 통과 완료). 이 CHECK 없이 guard 가 이 값을 보내면 RPC INSERT 가 CHECK 위반으로 **진단 적재 전체가 조용히 실패**한다 — 그래서 migration 을 먼저 적용하고 crawler guard(`e7240e3`, PR #14) 를 나중에 merge하는 순서 게이트를 지켰다.
- **한계**: 완전한 원자적 경합 해결이 아니다("실제 경합 축소"가 아니라 "이미 관측된 최신값보다 오래된 실행의 후행 write 차단"). 위 항목 3 참조.

## 구조

```
community-trend-crawler/
├── main.py              # 진입점, 스크래퍼 + 키워드 크롤러 통합 실행
├── models.py            # Post 데이터 모델
├── config.py            # 공통 설정 (Chrome 12종 헤더, 타임아웃 등)
├── scrapers/            # 커뮤니티 게시글 크롤러
│   ├── base.py          # BaseScraper (fetch, fetch_bytes, fetch_og_image)
│   ├── clien.py · ruliweb.py · ppomppu.py · mlbpark.py
│   ├── bobaedream.py · inven.py · dcinside.py · humoruniv.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kskedu/community-trend-crawler](https://github.com/kskedu/community-trend-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
