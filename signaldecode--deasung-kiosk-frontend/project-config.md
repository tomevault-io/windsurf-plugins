---
trigger: always_on
description: 1. **키오스크 앱과 어드민 웹을 한 레포에서 일관된 구조로 관리**
---

# CLAUDE.md (DAESUNG KIOSK Frontend Template)

## 핵심 목표 (절대 우선순위)

1. **키오스크 앱과 어드민 웹을 한 레포에서 일관된 구조로 관리**
2. **기능 추가보다 안정성, 예측 가능성, 유지보수성을 우선**
3. **UI / 도메인 로직 / API / 타입 / Electron 브리지 역할을 명확히 분리**
4. **장비 입력(카드/QR) 처리 흐름을 단순하고 안전하게 유지**
5. **공통 타입과 공통 모듈을 최대한 재사용**
6. **키오스크 특성상 전체화면, 고정 동선, 빠른 입력, 복구 용이성을 고려**
7. **불필요한 추상화보다 읽기 쉽고 수정 쉬운 구조를 우선**
8. **어드민과 키오스크는 같은 레포에 두되, 실행/배포 단위는 분리**

---

## 개발 사고 순서 (항상 이 순서로 설계/출력)

도메인 흐름 → 화면 목적 → 사용자 동선 → 상태 → API 계약 → 컴포넌트 구조 → 예외 처리 → 스타일 → 접근성

---

## 프로젝트 성격

이 프로젝트는 **학원/독서실/교육 공간용 키오스크 프론트엔드 시스템**이다.

구성:
- **admin-web**: 관리자용 웹 페이지
- **kiosk-app**: 현장 키오스크용 Electron 앱
- **shared**: 공통 타입, 공통 API 함수, 공통 상수/유틸

핵심 특징:
- 카드리더기 / QR리더기 입력으로 사용자 식별
- 출결, 좌석, 공지, 순공시간, 랭킹 등 기능 연동
- 키오스크는 빠른 입력과 최소한의 클릭으로 동작해야 함
- 관리자 페이지는 설정/조회/운영 중심

---

## 기술 스택

### admin-web
- React
- TypeScript
- Vite

### kiosk-app
- Electron
- React
- TypeScript
- Electron Forge
- Webpack 기반 템플릿

### 공통
- Backend: Java / Spring Boot / REST API
- 패키지 매니저: npm 또는 추후 pnpm 확장 가능
- 스타일: 프로젝트 진행 방향에 맞춰 CSS / SCSS / CSS Module 중 일관되게 선택
- 상태관리는 필요한 범위에서만 도입하며, 과도한 전역 상태 사용 금지

---

## 모노레포 구조 원칙

권장 구조:

project-root
├─ admin-web
├─ kiosk-app
├─ packages
│  ├─ shared-types
│  ├─ shared-api
│  ├─ shared-utils
│  └─ shared-constants
├─ docs
└─ CLAUDE.md

초기 단계에서는 `admin-web`, `kiosk-app` 중심으로 시작하고,
공통 코드가 명확해질 때만 `packages/*`로 분리한다.

### 원칙
- 레포는 하나로 관리
- 앱은 분리
- 배포도 분리
- 공통화는 필요할 때만 수행
- 너무 이른 shared 분리는 금지

---

## 아키텍처 원칙

### 1. 역할 분리
- **React Renderer**: 화면, 사용자 입력, 렌더링
- **Electron Main**: 창 제어, 앱 라이프사이클, OS 기능
- **Electron Preload**: renderer에 안전하게 노출할 브리지
- **API Layer**: 서버 통신
- **Domain Layer**: 출결/좌석/회원/공지 등 업무 로직
- **UI Layer**: 화면 요소 및 레이아웃

### 2. 입력 장치 처리
카드리더기 / QR리더기는 우선 아래 두 경우로 구분한다.

#### A. HID Keyboard 방식
- 리더기가 키보드처럼 문자열 입력
- React input 또는 전역 입력 버퍼로 처리 가능

#### B. Native / Serial / 별도 연동 방식
- Electron main 또는 preload를 통해 연동
- renderer는 직접 장치 접근하지 않음
- IPC 또는 contextBridge를 통해 최소 API만 사용

### 3. 보안
- renderer에서 Node API 직접 사용 금지
- preload를 통해 필요한 기능만 노출
- `ipcRenderer` 전체 노출 금지
- 민감한 시스템 기능은 main 프로세스에서 처리

---

## 도메인 설계 원칙

이 프로젝트의 주요 도메인은 아래와 같다.

- Branch(지점)
- KioskAuth(키오스크 지점 인증)
- Member(회원)
- Attendance(출결)
- Seat(좌석)
- Notice(공지)
- Ranking(순공시간 / 랭킹)
- DeviceInput(카드/QR 입력)
- ApprovalFlow(DSA 승인/거부 등 실시간 응답)

### 도메인 원칙
- 화면 이름보다 도메인 이름을 우선한다
- 문자열 상수는 하드코딩하지 말고 상수화한다
- API 응답 구조는 타입으로 명시한다
- 출결/좌석 상태값은 enum성 상수로 관리한다

예:
- `ATTENDANCE_STATUS`
- `SEAT_STATUS`
- `DEVICE_INPUT_TYPE`
- `KIOSK_MODE`

---

## 타입 설계 원칙 (TypeScript 필수)

이 프로젝트는 **JavaScript보다 TypeScript를 우선**한다.

### 이유
- admin-web / kiosk-app / shared 간 타입 공유 필요
- API 응답 구조가 명확해야 함
- Electron IPC payload 타입이 중요함
- 카드/QR 입력 데이터 구조를 안전하게 관리해야 함

### 원칙
- `any` 사용 금지
- API DTO 타입 정의
- UI Props 타입 정의
- IPC request / response 타입 정의
- nullable 여부를 명확히 표현

예시 타입 범주:
- `MemberSummary`
- `AttendanceRecord`
- `SeatInfo`
- `NoticeItem`
- `CardScanPayload`
- `QrScanPayload`
- `KioskBranchAuthResponse`

---

## 폴더 구조 규칙

### admin-web 권장 구조
admin-web/src
├─ app or pages
├─ components
├─ features
├─ entities
├─ shared
├─ api
├─ hooks
├─ utils
├─ types
└─ styles

### kiosk-app 권장 구조
kiosk-app/src
├─ main
├─ preload
├─ renderer
│  ├─ components
│  ├─ features
│  ├─ entities
│  ├─ pages
│  ├─ hooks
│  ├─ api
│  ├─ utils
│  ├─ types
│  └─ styles
└─ shared

### 역할
- `components`: 재사용 UI
- `features`: 사용자 행동 중심 기능
- `entities`: 도메인 단위 모델/뷰
- `pages`: 화면 단위
- `api`: 서버 통신
- `types`: 타입 정의
- `utils`: 순수 유틸
- `hooks`: 재사용 훅

---

## 네이밍 규칙

### 파일명
- React 컴포넌트: PascalCase
- 훅: `useSomething.ts`
- 유틸: camelCase
- 상수: `UPPER_SNAKE_CASE`
- 타입: PascalCase
- Electron IPC 채널명: 도메인 기반 소문자 문자열

예:
- `AttendancePage.tsx`
- `SeatStatusCard.tsx`
- `useCardScanner.ts`
- `attendanceApi.ts`
- `seatConstants.ts`

### IPC 채널명 규칙
형식:
`domain:action`

예:
- `card:scanned`
- `qr:scanned`
- `kiosk:login`
- `window:reload`
- `device:status`

---

## UI / 화면 설계 원칙

### 키오스크 화면
키오스크는 일반 웹이 아니라 **현장 장치 UI**다.

반드시 고려:
- 버튼 수 최소화
- 클릭 동선 최소화
- 큰 텍스트 / 큰 터치 영역
- 현재 상태를 즉시 이해 가능해야 함
- 잘못된 입력 후 복구가 쉬워야 함
- 제한 시간/자동 초기화 필요 여부를 항상 고려

### 어드민 화면
- 운영자가 정보 조회/설정/관리하기 쉬운 정보 구조
- 목록 / 상세 / 검색 / 필터 / 상태 표시를 명확하게 구성
- CRUD보다 “운영 흐름”이 드러나게 설계

---

## 스타일 원칙

- 일관성 없는 스타일 혼용 금지
- 인라인 스타일 금지
- 컴포넌트 내부에 과도한 스타일 하드코딩 금지
- 공통 색상 / spacing / radius / shadow / z-index는 토큰화 권장
- 키오스크 화면은 시인성과 터치 친화성을 우선
- 어드민은 정보 밀도와 가독성 균형 우선

### 권장 방향
- 초기에는 단순한 구조 유지
- 디자인 시스템은 필요한 만큼만
- 너무 이른 추상화 금지
- 공통 버튼/모달/배지 정도부터 재사용

---

## 상태관리 원칙

- 로컬 상태로 충분하면 local state 사용
- 전역 상태는 정말 필요한 경우만 사용
- 서버 상태와 UI 상태를 구분
- 일시적인 입력 상태를 전역으로 올리지 말 것

예:
- 입력 중인 카드값 → 로컬 상태
- 현재 로그인된 지점 정보 → 전역 가능
- 서버에서 가져온 공지 목록 → 서버 상태로 관리
- 좌석 선택 중 임시 값 → 로컬 상태 우선

---

## API 연동 원칙

Backend는 Spring REST API를 기본으로 한다.

### 기본 원칙
- API 경로는 하드코딩하지 말고 상수/설정으로 분리
- fetch/axios 사용 시 공통 클라이언트 계층 구성 가능
- 요청/응답 타입 정의
- 에러 응답 구조 정리
- UI 컴포넌트 안에서 직접 API 세부 구현 남발 금지

### 예시 엔드포인트 범주
- `POST /api/v1/kiosk/login`
- `POST /api/v1/kiosk/card-scan`
- `POST /api/v1/kiosk/qr-scan`
- `GET /api/v1/notices`
- `GET /api/v1/rankings`
- `GET /api/v1/seats`
- `POST /api/v1/attendance/check-in`
- `POST /api/v1/attendance/check-out`

실제 엔드포인트는 백엔드 계약을 기준으로 맞춘다.

---

## 장비 입력 처리 원칙

### 카드/QR 입력 흐름
1. 장비 입력 수신
2. 문자열 정규화(trim, suffix 제거 등)
3. 입력 타입 식별(card / qr)
4. 중복 입력 방지 처리
5. 서버 조회 또는 로컬 검증
6. 성공/실패 UI 피드백
7. 일정 시간 후 초기 상태 복귀

### 원칙

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SIGNALDECODE) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
