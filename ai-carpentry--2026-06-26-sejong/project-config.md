---
trigger: always_on
description: 세종대학교 2026-06-26 워크숍 사례로 정리한 The Carpentries 공식 `workshop-template` 기반 웹사이트 구축 절차.
---

# Software Carpentry 워크숍 웹사이트 구축 가이드

세종대학교 2026-06-26 워크숍 사례로 정리한 The Carpentries 공식 `workshop-template` 기반 웹사이트 구축 절차.

---

## 1. 개요

- **템플릿 출처**: https://github.com/carpentries/workshop-template
- **배포**: GitHub Pages (`gh-pages` 브랜치)
- **정적 사이트 생성기**: Jekyll (github-pages gem 사용)
- **저장소 명명 규칙**: `YYYY-MM-DD-사이트명` (예: `2026-06-26-sejong`)

---

## 2. 사전 요구사항

| 도구 | 최소 버전 | 비고 |
|------|----------|------|
| Ruby | 3.0+ | `ffi` gem 호환을 위해 2.7은 권장하지 않음 |
| Bundler | 2.x | `gem install bundler` |
| Git | 2.x | |
| GitHub 계정 | — | `gh-pages` 브랜치 자동 배포 |
| Eventbrite 계정 | — | 참가 등록 (선택) |

rbenv 사용자는 `.ruby-version` 확인 후 필요 시 최신 버전 설치.

---

## 3. 저장소 초기 세팅

### 3.1 템플릿에서 새 저장소 생성
1. https://github.com/carpentries/workshop-template 에서 **Use this template** → 저장소명을 `YYYY-MM-DD-장소` 형식으로 생성 (예: `2026-06-26-sejong`).
2. 로컬 클론:
   ```bash
   git clone git@github.com:<org>/2026-06-26-sejong.git
   cd 2026-06-26-sejong
   ```
3. 기본 브랜치를 `gh-pages`로 설정 (GitHub Pages가 이 브랜치에서 자동 빌드).

### 3.2 Bundler 설치
```bash
bundle config set --local path .vendor/bundle
bundle install
```

**Ruby 버전 호환 문제 발생 시** (예: `ffi requires ruby >= 3.0`):
```bash
# 현재 플랫폼을 lock 파일에 추가한 뒤 호환 버전으로 다운그레이드 재해결
bundle lock --add-platform x86_64-darwin
bundle lock --update=ffi
bundle install
```
근본 해결은 Ruby 3.0 이상 설치.

---

## 4. `index.md` front matter 구성

워크숍 정보는 `index.md` 상단 YAML front matter에 기술한다. 세종대 사례:

```yaml
---
layout: workshop                                           # 고정값
venue: "Sejong University"                                 # 기관명 (주소 제외)
address: "서울특별시 광진구 능동로 209(군자동) 율곡관 201호"  # 상세 주소
country: "kr"                                              # ISO 3166-1 alpha-2 소문자
language: "ko"                                             # ISO 639-1 소문자
latitude: "37.550106"                                      # https://www.latlong.net/
longitude: "127.073171"
humandate: "Jun 26-27, 2026"                               # ⚠ 3글자 월 형식 필수
humantime: "10:00 am - 5:00 pm KST (1:00 am - 8:00 am UTC)"
startdate: 2026-06-26                                      # ISO 8601
enddate: 2026-06-27
instructor: ["Kwangchun Lee"]
helper: ["HwanHee Hyung"]
email: ["kwangchun.lee.7@gmail.com"]
collaborative_notes: https://pad.carpentries.org/2026-06-26-sejong
eventbrite: "1988050060247"                                # 문자열로 감싸기
---
```

### 4.1 주의할 필드

- **`humandate`**: `make workshop-check`가 **3글자 월 + 4자리 연도** 형식만 통과시킴. `"June 26-27, 2026"` → **실패**, `"Jun 26-27, 2026"` → 통과.
- **`humantime`**: KST 기준과 UTC 환산을 병기. KST = UTC+9.
- **`language`**: 한국어 페이지는 `"ko"`로 지정. 하지만 기본 `_layouts/workshop.html`은 `<html lang="en">`이 하드코딩되어 있어 별도 수정 필요 (§7 참고).
- **`eventbrite`**: 숫자형으로 두면 Liquid가 정수로 처리할 수 있으므로 **반드시 따옴표로 감쌀 것**.

---

## 5. Eventbrite 등록 연동

### 5.1 Eventbrite 이벤트 생성
1. https://eventbrite.com → **Create Event**
2. 기본 정보 입력:
   - 이벤트명: `Software Carpentry Workshop at Sejong University`
   - 날짜/시간: 워크숍 일정 (KST)
   - 장소: 대면이면 실주소, 온라인이면 URL
3. 티켓 설정 (무료/유료, 정원 등) 후 **Publish**.
4. 이벤트 URL 끝자리의 숫자 ID 복사 (예: `https://www.eventbrite.com/e/...-1988050060247` → `1988050060247`).
5. `index.md` front matter에 `eventbrite: "1988050060247"` 입력.

### 5.2 ⚠ 레거시 iframe 엔드포인트 문제

`workshop-template` 기본값은 다음 iframe을 렌더링:
```html
<iframe src="https://www.eventbrite.com/tickets-external?eid=...&ref=etckt">
```

**문제**: 이 `tickets-external` 경로는 deprecated. Eventbrite의 CloudFront WAF가 리퍼러·지역·브라우저 조건에 따라 **403 Forbidden**을 반환하는 사례 빈번. 한국 IP에서 특히 잦음.

**해결**: `index.md`의 iframe 블록을 공식 최신 위젯 + 직접 링크 fallback으로 교체.

```html
{% if page.eventbrite %}
<div id="eventbrite-widget-container-{{page.eventbrite}}" style="margin: 1em 0;"></div>
<p class="text-center">
  <a id="eventbrite-widget-modal-trigger-{{page.eventbrite}}"
     href="https://www.eventbrite.com/e/{{page.eventbrite}}"
     class="btn btn-success btn-lg"
     target="_blank" rel="noopener">
    Register on Eventbrite
  </a>
</p>
<script src="https://www.eventbrite.com/static/widgets/eb_widgets.js"></script>
<script type="text/javascript">
  window.EBWidgets && window.EBWidgets.createWidget({
    widgetType: 'checkout',
    eventId: '{{page.eventbrite}}',
    iframeContainerId: 'eventbrite-widget-container-{{page.eventbrite}}',
    iframeContainerHeight: 425,
    modal: true,
    modalTriggerElementId: 'eventbrite-widget-modal-trigger-{{page.eventbrite}}'
  });
</script>
{% endif %}
```

아울러 `_includes/javascript.html`에서 **구식 iframe 높이 검사 훅도 제거** — 더 이상 의미 없음.

---

## 6. 스케줄 편집

`index.md` 하단의 Day 1 / Day 2 테이블을 실제 커리큘럼에 맞게 편집. 세종대 10:00-17:00 사례:

```html
<tr> <td>10:00</td>  <td>Automating Tasks with the Unix Shell</td> </tr>
<tr> <td>11:30</td>  <td>Morning break</td> </tr>
<tr> <td>12:00</td>  <td>Automating Tasks with the Unix Shell (Continued)</td> </tr>
<tr> <td>13:00</td>  <td>Lunch break</td> </tr>
<tr> <td>14:00</td>  <td>R for Reproducible Scientific Analysis</td> </tr>
<tr> <td>15:30</td>  <td>Afternoon break</td> </tr>
<tr> <td>16:00</td>  <td>R for Reproducible Scientific Analysis (Continued)</td> </tr>
<tr> <td>16:30</td>  <td>Wrap-up</td> </tr>
<tr> <td>17:00</td>  <td>END</td> </tr>
```

`humantime`의 시작·종료 시각과 **스케줄 테이블의 마지막 END 행을 반드시 일치**시킬 것.

---

## 7. 한국어 페이지 i18n 패치


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-carpentry/2026-06-26-sejong](https://github.com/ai-carpentry/2026-06-26-sejong) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
