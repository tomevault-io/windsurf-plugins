---
trigger: always_on
description: 이 파일은 Claude Code가 매 세션 시작 시 가장 먼저 읽고 프로젝트 맥락을 파악하는 자급자족형 지침이다.
---

# 풍당수학 학원 관리 시스템 — CLAUDE.md

이 파일은 Claude Code가 매 세션 시작 시 가장 먼저 읽고 프로젝트 맥락을 파악하는 자급자족형 지침이다.
세션 마무리 시 Claude Code가 이 파일을 직접 갱신하여 다음 세션으로 컨텍스트를 넘긴다.

---

## 1. 앱 기본 정보

- **앱 이름**: 풍당수학 학원 관리 시스템
- **현재 파일**: `index.html` (단일 HTML 파일)
- **GitHub 저장소**: https://github.com/beybusiness-bit/pongdang-manager
- **배포 URL**: https://beybusiness-bit.github.io/pongdang-manager/

```javascript
const AUTH = {
  ALLOWED_EMAILS: ['pongdang.math@gmail.com'],
  CLIENT_ID: 'TBD',
  SHEET_ID: '1_CFlWnHOi13EFlp8QAUYATdo6lruzXMDfAelxR1Xa0o',
};
```

---

## 2. 앱 아키텍처 요약

- **앱 성격**: 풍당수학 학원장과 강사들이 학생·결제·교재·근무·매출을 한 곳에서 관리하는 내부 운영 도구. 기존에 노션과 구글 시트로 분산해 쓰던 자료를 단일 앱으로 통합 마이그레이션.
- **UI 구조**: 좌측 사이드바 + 메인 콘텐츠. 사이드바 메뉴: 대시보드 / 학생 관리 / 교재 관리 / 결제 관리(서브탭: 결제기록·월별등록대장) / 캘린더 / 리포트(매출 리포트 등). 데스크탑 우선, 일부 페이지는 모바일 비대응 안내.
- **로그인**: Google OAuth 필수. `ALLOWED_EMAILS`에 등록된 계정만 진입 가능.
- **사용자 역할**: 학원장(OWNER) / 선생님(TEACHER) / 보조선생님(ASSISTANT) — 역할별로 세부 권한(보기·쓰기) 분기. 현재 더미 사용자는 학원장 역할.
- **외부 연동**: Google OAuth (로그인) + Google Sheets API (데이터 저장). 추후 문자 CRM, 이메일 등 연동 예정.
- **기술 스택**: 단일 HTML 파일 + 인라인 CSS/JS, GitHub Pages 배포

---

## 3. 세션 운영 원칙

### 세션 시작 시
Claude Code는 **반드시 이 파일을 먼저 읽고** 다음을 파악한 뒤 작업을 시작한다:
- "개발 단계 현황"에서 현재 어느 phase에 있고 무엇이 완료되었는지
- "다음 세션 시작점"에서 당장 이어서 할 작업이 무엇인지
- "가이드 문서 참고 내용 누적"에서 누적된 UX·입력 규칙·주의사항

### 용어 구분 (중요)
- **Phase**: "개발 단계 현황"에 나열된 큰 개발 단위 (예: "결제 관리", "근무일지" 등)
- **작업 단계**: 한 세션 안에서 일을 쪼개서 진행하는 하위 단위
- ⚠️ 둘을 절대 혼동하지 말 것. "작업 단계 하나 끝났다"는 이유로 마무리 작업(이 파일 갱신)을 수행하면 안 된다.

### 새 세션 권유 (Claude Code가 먼저 제안 가능)
다음 경우 사용자에게 새 세션을 권유할 수 있다:
- 한 phase가 완전히 마무리되었을 때
- 컨텍스트가 과도하게 쌓여 작업 효율이 떨어질 때
- 새로운 기능 영역에 진입할 때
- 오류 반복으로 맥락이 꼬였을 때

권유는 권유일 뿐. 사용자의 응답을 기다린다.

### CLAUDE.md 갱신 트리거 (사용자의 명시적 의사 표현 필수)
이 파일의 갱신은 **오직 사용자가 명시적으로 세션 종료 의사를 표현할 때만** 수행:
- "마무리할게" / "세션 끝내자" / "새 세션 열게" 등
- Claude Code의 권유에 사용자가 "응" / "그래" / "그러자" 등으로 동의

❌ **절대 금지**:
- 작업 단계 하나가 끝났다는 이유로 자동 갱신
- Claude Code가 권유한 직후 사용자 응답 없이 자동 갱신
- 사용자 피드백/테스트 결과 받기 전 자동 갱신

### 갱신 시 반영 대상
- 완료된 phase는 ✅로 표시
- "다음 세션 시작점" 업데이트
- DB 구조 변경 사항 반영
- 이번 세션에서 가이드 문서에 참고할 만한 내용을 "가이드 문서 참고 내용 누적"에 추가
- ⚠️ 원본 내용을 임의로 축약·삭제하지 말 것. 사용자와의 논의로 합의된 변경만 수행.

---

## 4. Phase 2: 개발 진행 프로토콜

### 코드 작업 원칙

```javascript
// today() — 반드시 로컬 날짜 기준 (toISOString() UTC 방식 절대 금지)
const today = () => {
  const d = new Date();
  return d.getFullYear() + '-'
    + String(d.getMonth() + 1).padStart(2, '0') + '-'
    + String(d.getDate()).padStart(2, '0');
};
```

- 파일 상단의 `const AUTH = {...}` 상수는 매 작업 후에도 위 실제값을 유지한다.
- 작동하는 코드는 함부로 전체 교체하지 않는다. 부분 수정 우선.
- 사용자가 명시적으로 지시하지 않은 변경·추가 구현을 임의로 진행하지 않는다. 의문이 생기면 반드시 확인.

### 절대 금지 사항
- `<script>` 내부 백틱(`) 중첩
- `<script>` 내부 `&` `<` `>` `"` 직접 삽입 (HTML 파싱 깨짐 위험)
- `innerHTML` 사용 시 null 체크 생략
- `toISOString()` 기반 UTC 날짜 사용
- `</script>` 바깥에 코드가 빠져나오는 구조 (과거 버그 사례 있음 — 반드시 `</script>` 안쪽에서 함수 정의)
- 같은 함수 이름의 중복 정의 (뒤의 정의가 앞을 덮어쓰는 사고 발생함)

### 매 작업 후 필수
- `node --check` 또는 동등한 방식으로 script 블록 문법 검사 (HTML 파일이라 `node`로 직접은 안 되니, script 태그 내부만 추출해서 `new Function(scriptContent)` 검증)
- 사용자가 결과물을 받아볼 수 있는 형태로 전달

### 수정 방식
- 부분 수정: `Edit` 또는 `str_replace` 활용
- 대량 교체(50줄 이상): Python 스크립트로 라인 범위 교체
- 새 함수 추가: 관련 함수 근처에 추가하되, 반드시 `</script>` 안쪽인지 확인

### 모달 중첩 / z-index
- 현재 `openModal()` 함수가 자동으로 z-index를 누적 계산 (1000부터 +10씩)
- 모달 위에 모달 띄우기 가능
- `closeModal()`은 300ms setTimeout으로 display:none 처리하므로, 닫고 바로 다른 모달 열 때는 100ms 정도 setTimeout 권장

---

## 5. Phase 3: 배포 프로토콜

1. `index.html` GitHub 저장소 (https://github.com/beybusiness-bit/pongdang-manager) 에 업로드
2. GitHub Repository → Settings → Pages → Branch: `main` → Save
3. Google Cloud Console → OAuth 클라이언트 ID에 배포 URL(`https://beybusiness-bit.github.io/pongdang-manager/`) 추가
4. ALLOWED_EMAILS 계정들이 OAuth 동의 화면 테스트 사용자로 등록되어 있는지 확인
5. 배포 후 ALLOWED_EMAILS 계정으로 로그인 테스트

빠른 테스트 옵션:
- UI만 빠르게 보고 싶다 → Chrome으로 index.html 직접 열기
- Sheets 포함 전체 동작 → netlify.com/drop 에 드래그 앤 드롭으로 임시 배포
- 최종 → GitHub Pages

---

## 6. Phase 4: 가이드 문서 작성 프로토콜

배포 완료 후 사용자 요청 시, 아래 프롬프트를 완성해서 출력한다.
이 시점에서 "가이드 문서 참고 내용 누적" 항목의 내용을 프롬프트 안에 포함시킨다.

```
지금까지 이 프로젝트에서 개발한 앱의 사용 가이드를 노션에 작성해줘.
노션 MCP로 바로 작성. 작성할 노션 페이지 URL: [URL]

앱 이름: 풍당수학 학원 관리 시스템
접속 URL: https://beybusiness-bit.github.io/pongdang-manager/
로그인: Google OAuth (허가된 이메일만)
허가 이메일: pongdang.math@gmail.com (외 추가된 계정)
데이터 저장: Google Sheets (ID: 1_CFlWnHOi13EFlp8QAUYATdo6lruzXMDfAelxR1Xa0o)

주요 기능 (사이드바 순서): [구현된 메뉴 전체]
주요 워크플로우: [반복 업무 흐름]

개발 과정에서 수집된 가이드 참고 내용:
[아래 "가이드 문서 참고 내용 누적" 항목 전체 삽입]

문서 요건:
- 대상: 기술 배경 없는 처음 사용자
- 구성: 시작하기 → 메뉴별 기능 → 주요 워크플로우 → 주의사항·FAQ
- 각 섹션에 "📸 이미지 추가 위치: [캡처할 화면]" 안내 포함
- 관리자(학원장)용 섹션 포함
```

---

## 7. 개발 단계 현황

✅ **1단계: 기본 구조 및 로그인**
- Google OAuth 로그인
- 권한 시스템 (학원장 / 선생님 / 보조쌤)
- 세부 권한 (보기·쓰기)
- 기본 레이아웃 (사이드바, 헤더)

✅ **2단계: Google Sheets 연동 및 데이터 구조**
- 20개 시트 생성 및 스키마 정의
- Sheets API 연동
- 기본 CRUD 함수

✅ **3단계: 대시보드**
- 학생 수, 매출, 공지 등 간소화된 대시보드
- 공휴일 표시

✅ **4단계: 학생 관리 기본**
- 학생 목록 (필터링, 검색)
- 학생 추가/수정/삭제
- [개선] 학생 추가/수정 모달이 콜백 지원 (월별등록대장에서 재사용 가능)
- [개선] 행 전체 클릭 시 상세 모달 오픈
- [개선] "작업" 컬럼 제거
- [개선] 필터 헤더 '학년' → '학교급' 교정
- [개선] 테이블 헤더/셀 중앙 정렬

✅ **5단계: 학생 관리 고급**
- 학생 상세 페이지 (모달)
- 권한별 데이터 접근 제어
- [개선] 상세 모달의 "수정" 버튼 정상 동작 (중복 함수 제거 + setTimeout 딜레이)

✅ **6단계: 교재 관리**
- 교재 목록
- 교재 CRUD
- 학생-교재 연결
- 프로필 사진 시스템 (Base64)
- 등록/퇴원 기록

🔄 **7단계: 결제 관리** (진행 중)

✅ 완료된 부분:
- 결제 내역 입력 (개별)
- 결제기록 모달 UI 개선
- 요일 양방향 연동 (결제기록 ↔ 학생 관리)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beybusiness-bit/pongdang-manager](https://github.com/beybusiness-bit/pongdang-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
