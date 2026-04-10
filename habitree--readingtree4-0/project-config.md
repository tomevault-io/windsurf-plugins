---
trigger: always_on
description: **프로젝트:** Habitree Reading Hub v4.0.0
---


## [Data Model Governance]

# 프로젝트 데이터 구조 규칙 (Data Model Governance)

**작성일:** 2025년 1월  
**프로젝트:** Habitree Reading Hub v4.0.0  
**버전:** 2.0  
**적용 범위:** 전체 프로젝트

**참고:** 
- 레이어 분리 상세 규칙은 `.cursor/rules/ui_datarule.mdc`를 참조하세요.
- DB/RLS 관리 규칙은 `.cursor/rules/db_rls_rule.mdc`를 참조하세요.
- 마이그레이션 관리 규칙은 `.cursor/rules/migration_rule.mdc`를 참조하세요.

---

## 목차

1. [규칙 개요](#1-규칙-개요)
2. [레이어 분리 원칙](#2-레이어-분리-원칙)
3. [데이터 모델 단일 기준](#3-데이터-모델-단일-기준)
4. [스키마 변경 규칙](#4-스키마-변경-규칙)
5. [데이터 무결성 원칙](#5-데이터-무결성-원칙)
6. [타입 안정성](#6-타입-안정성)
7. [변경 사항 문서화](#7-변경-사항-문서화)
8. [점진적 마이그레이션 전략](#8-점진적-마이그레이션-전략)
9. [필수 폴더 및 파일 구조](#9-필수-폴더-및-파일-구조)
10. [규칙 위반 시 조치](#10-규칙-위반-시-조치)
11. [실제 적용 예시](#11-실제-적용-예시)

---

## 1. 규칙 개요

### 1.1 규칙의 목적

이 프로젝트는 **Next.js + Supabase 기반**이며, **Vercel에 배포**됩니다.  
개발 방식은 **바이브코딩**이므로, 데이터 구조와 DB 접근이 무너지지 않도록 **엄격한 구조 규칙**이 반드시 필요합니다.

### 1.2 규칙의 핵심 원칙

- ✅ **단일 기준 문서**: 모든 데이터 구조는 `doc/database/DATA_MODEL.md`에 정의
- ✅ **레이어 분리**: 컴포넌트는 UI만, DB 접근(쿼리 실행) 은 `app/actions/`에서만
- ✅ **DB 구조 노출 금지**: UI 레이어(컴포넌트)에서 테이블명, 컬럼명 등 DB 구조 노출 금지
- ✅ **타입 안정성**: 모든 DB 접근은 타입을 명시
- ✅ **문서화 필수**: 스키마 변경 시 반드시 문서 업데이트
- ✅ **점진적 적용**: 기존 코드는 유지, 새 코드부터 엄격히 적용

---

## 2. 레이어 분리 원칙

### 2.1 컴포넌트의 책임

컴포넌트(`.tsx`, `.jsx`)는 다음을 **절대 수행하면 안 됩니다**:

```typescript
// ❌ 금지: 컴포넌트에서 Supabase 직접 호출
import { createClient } from "@/lib/supabase/client";

export function BookList() {
  const supabase = createClient();
  const { data } = await supabase.from("user_books").select("*"); // 금지!
  return <div>...</div>;
}

// ❌ 금지: UI 레이어에서 DB 구조 노출 (테이블명, 컬럼명 등)
const tableName = "user_books"; // 금지!
const columnName = "user_id"; // 금지!
if (book.user_id === user.id) { ... } // 권한 판단 로직도 금지!
```

### 2.2 컴포넌트가 해야 하는 것

컴포넌트는 다음만 수행합니다:

- ✅ 화면 렌더링
- ✅ 사용자 입력 처리
- ✅ `hooks/`를 통한 데이터 접근 (표준 패턴)

```typescript
// ✅ 올바른 예: hooks를 통한 간접 호출 (표준 패턴)
import { useBooks } from "@/hooks/use-books";

export function BookList() {
  const { books, isLoading, error } = useBooks();
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      {books.map(book => (
        <BookCard key={book.id} book={book} />
      ))}
    </div>
  );
}
```

### 2.3 데이터 접근 레이어 구조

데이터 접근은 다음 계층 구조를 따릅니다:

```
컴포넌트 (UI)
  ↓
hooks/ (상태 관리 및 비동기 처리)
  ↓
app/actions/ (Server Actions - DB 접근)
  ↓
Supabase
```

**표준 패턴:**
- 컴포넌트 → `hooks/` → `app/actions/` → Supabase

### 2.4 허용된 데이터 접근 경로

Supabase와의 모든 통신은 **오직 아래 경로에서만** 허용됩니다:

1. **`app/actions/**`** (Next.js Server Actions - **유일한 DB 접근 레이어**)

### 2.5 예외 사항 (인증 관련)

다음은 예외로 허용됩니다:

- ✅ `lib/supabase/client.ts`, `lib/supabase/server.ts` (클라이언트 생성 유틸리티)
- ✅ `contexts/auth-context.tsx` (인증 상태 관리)
- ✅ `app/callback/route.ts` (OAuth 콜백 처리)

```typescript
// ✅ 허용: 인증 컨텍스트에서의 Supabase 호출
// contexts/auth-context.tsx
const supabase = createClient();
const { data: { user } } = await supabase.auth.getUser(); // 허용
```

### 2.6 Server Actions 규칙

`app/actions/` 내의 Server Actions는:

- ✅ Supabase 호출을 직접 수행할 수 있음
- ✅ 테이블명, 컬럼명은 문자열 리터럴로 사용 가능 (Server Actions 내부이므로)
- ✅ 모든 DB 접근 로직은 이 레이어에 집중
- ✅ 권한 검증 및 RLS 정책을 신뢰하되, 필요한 경우 추가 검증 수행
- ✅ 반환 타입은 반드시 명시

```typescript
// ✅ 올바른 예: Server Action에서 타입 명시
"use server";
import { createServerSupabaseClient } from "@/lib/supabase/server";
import type { Database } from "@/types/database";

export async function getUserBooks() {
  const supabase = await createServerSupabaseClient();
  
  const { data, error } = await supabase
    .from("user_books")
    .select("*")
    .returns<Database["public"]["Tables"]["user_books"]["Row"][]>();
  
  if (error) throw error;
  return data || [];
}
```

---

## 3. 데이터 모델 단일 기준

### 3.1 단일 기준 문서

데이터베이스 구조의 **단일 기준 문서**는 하나만 존재해야 합니다:

**`doc/database/DATA_MODEL.md`**

### 3.2 필수 정의 사항

모든 데이터 관련 개념은 반드시 이 문서에 정의되어야 합니다:

- ✅ 테이블
- ✅ 컬럼
- ✅ 테이블 간 관계
- ✅ 소유 구조(사용자/그룹)
- ✅ 접근 제어 원칙(RLS 의도)

### 3.3 테이블 정의 요구사항

어떤 테이블도 다음 정보 없이 존재할 수 없습니다:

1. **한 문장으로 된 목적 정의**
2. **소유자/권한 기준** (auth.users 기준)
3. **관계 정의** (외래 키)

### 3.4 중복 테이블 금지

동일한 개념을 표현하는 테이블이 둘 이상 존재하는 것은 **금지**합니다.

```sql
-- ❌ 금지: 중복 테이블
CREATE TABLE user_books (...);
CREATE TABLE user_bok2 (...); -- 같은 개념의 중복 테이블 금지!
```

### 3.5 임시/실험용 테이블 관리 규칙

임시 또는 실험용 테이블은 다음 규칙을 따라야 합니다:

- ✅ **명명 규칙**: `_temp_<기능명>` 또는 `_experimental_<기능명>` 접두사 필수
- ✅ **문서화**: `DATA_MODEL.md`에 "임시/실험용" 표시 및 목적 명시
- ✅ **정리 조건**: 프로덕션 배포 전 반드시 제거 또는 정식 테이블로 전환
- ✅ **생명주기**: 생성일로부터 3개월 이내 정리 또는 전환

```sql
-- ✅ 허용: 명명 규칙 준수
CREATE TABLE _temp_search_cache (...);
CREATE TABLE _experimental_ai_suggestions (...);
```

---

## 4. 스키마 변경 규칙

### 4.1 변경 절차

데이터베이스 구조 변경이 발생하면 반드시 다음 절차를 따릅니다:

1. **`doc/database/DATA_MODEL.md`를 먼저 수정**
2. **실제 스키마 변경이 있다면 SQL 마이그레이션 파일 생성**

### 4.2 마이그레이션 파일 위치

SQL 마이그레이션 파일은 반드시 다음 경로에 생성합니다:

**`doc/database/`** (향후 `migrations/` 폴더로 정리 예정)

### 4.3 파일명 규칙

```
migration-YYYYMMDDHHmm__<기능명>__<변경내용>.sql
```

**예시:**
- `migration-202412151430__notes__add_page_number_column.sql`
- `migration-202412151500__books__add_isbn_index.sql`
- `migration-202412151600__users__update_rls_policy.sql`

### 4.4 스키마 변경 범위

스키마 변경에는 다음이 포함됩니다:

- 테이블 생성/삭제
- 컬럼 추가/삭제
- 인덱스 변경
- RLS 정책 변경
- 트리거 또는 함수 추가

### 4.5 Idempotent 작성 원칙

마이그레이션 파일은 **idempotent**하게 작성합니다:

```sql
-- ✅ 올바른 예: Idempotent 마이그레이션
CREATE TABLE IF NOT EXISTS notes (

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/habitree) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
