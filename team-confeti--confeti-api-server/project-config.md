---
trigger: always_on
description: - `api`: Controller, DTO, Facade를 포함하며 클라이언트 요청을 처리하는 프레젠테이션 계층
---

# 구조 및 설계 컨벤션

## 1. 패키지 구조

### 기본 구조

- `api`: Controller, DTO, Facade를 포함하며 클라이언트 요청을 처리하는 프레젠테이션 계층
    - `controller`: RESTful API 엔드포인트 정의
    - `dto`: Request/Response DTO 정의
    - `facade`: 여러 도메인 서비스를 조율하는 Facade 클래스
- `domain`: 도메인별로 구성되며 각 도메인은 다음의 하위 패키지를 가짐
    - Entity 클래스 (도메인 모델)
    - `application`: Service 클래스와 DTO 정의
    - `infra`: JPA Repository 구현체 및 외부 시스템 연동
- `auth`: 인증/인가 관련 로직 (JWT, OAuth, 로그인/로그아웃 등)
- `external`: 외부 API 클라이언트 (Feign Client 등)
- `global`: 공통 설정, 예외 처리, 유틸리티, 어노테이션 등

### 규칙

- **서비스 간 참조 금지**: 도메인 Service는 다른 도메인의 Service를 직접 참조하지 않음
- **Facade 계층 필수**: 여러 도메인 Service를 사용해야 하는 비즈니스 로직은 Facade 계층에서 처리
- 하위 모듈이 많을 경우 bounded context별로 패키지 분리

## 2. Facade 패턴

### 목적

도메인별로 Service를 나누면서 발생하는 서비스 간 참조 문제를 방지하기 위해 Facade 계층을 도입함.

### 역할

- 여러 도메인의 Service를 조율하여 복잡한 비즈니스 로직 처리
- Controller와 Service 사이의 중간 계층으로 동작
- 트랜잭션 경계 설정 (필요 시 `@Transactional` 적용)

### 구현 규칙

- `@Facade` 어노테이션 사용
- `api/{domain}/facade` 패키지에 위치
- 여러 도메인의 Service를 주입받아 사용 가능
- DTO 변환 로직 포함 가능

### 예시

```java

@Facade
@RequiredArgsConstructor
public class SetlistFacade {

    private final SetlistService setlistService;
    private final SetlistEditService setlistEditService;
    private final UserService userService;  // 다른 도메인의 Service 참조

    public SetlistCreateResponseDTO createSetLists(Long userId,
        List<SetlistCreateRequestDTO> requests) {
        User user = userService.findById(userId);  // UserService 사용
        return new SetlistCreateResponseDTO(
            setlistService.createSetLists(user, requests)  // SetlistService 사용
        );
    }
}
```

## 3. 도메인 모델 설계

### Entity 설계

- JPA Entity로 도메인 모델 구현
- 불변성과 캡슐화를 중심으로 설계
- Setter 사용 최소화 (필요한 경우에만 `@Setter` 명시 - 사용 이유 주석 필수)
- Builder 패턴 및 정적 팩토리 메서드 사용 권장 (`@Builder`)
- 생성일자 (`created_at`), 수정일자 (`updated_at`) 추가 필수
    - JpaAuditing 사용

### 테이블 네이밍

- 복수형 사용 (예: `users`, `concerts`, `festivals`)
- 스네이크 케이스(snake_case) 적용: `@Table(name = "table_name")`

### 컬럼 네이밍

- 스네이크 케이스 적용 (예: `created_at`, `user_id`)
- 불리언: `is_` 접두사 사용 불필요 (JPA는 필드명 그대로 매핑)
- Enum: `@Enumerated(EnumType.STRING)` 사용 권장

### ID 생성 전략

- `@GeneratedValue(strategy = GenerationType.IDENTITY)` 사용

### 연관관계

- 지연 로딩 우선 사용: `fetch = FetchType.LAZY`
- 양방향 연관관계 시 연관관계 편의 메서드 작성
- Cascade 타입은 신중하게 설정

### 감사(Auditing) 필드

- `@CreatedDate`, `@LastModifiedDate` 사용
- `@EntityListeners(AuditingEntityListener.class)` 적용

## 4. Repository 네이밍 규칙

### 조회(Query)

- `find`를 기본으로 사용 (예: `findById`, `findAllByUserId`)
- 단건 조회: `findBy...` 형태로 `Optional` 반환
- 다건 조회: `findAllBy...` 또는 `findBy...` 형태로 `List` 반환
- 존재 여부 확인: `existsBy...` 형태로 `boolean` 반환
- 카운트: `countBy...` 형태로 `long` 반환

### 저장, 수정, 삭제(Command)

- 저장: `save` 사용
- 수정: JPA의 Dirty Checking 활용 (별도 메서드 불필요)
- 삭제: `delete` 또는 `deleteById` 사용

### Repository 구성

- Repository는 인터페이스로 정의하며 `JpaRepository` 상속
- 커스텀 쿼리는 `@Query` 어노테이션 사용
- Repository는 `domain/{domain}/infra/repository` 패키지에 위치

## 5. Service 네이밍 규칙

### 조회(Query)

- `get` 사용을 권장 (예: `getUserInfo`, `getSetlistDetail`)
- null 체크와 예외 처리 필수
    - 예: `repository.findById(id).orElseThrow(() -> new NotFoundException(ErrorMessage.NOT_FOUND))`
- 리스트 조회: `getAll...` 형태 사용 (예: `getAllMySetlists`)

### 저장, 수정, 삭제(Command)

- 생성: `create` 사용 (예: `createSetList`)
- 수정: `update` 또는 `patch` 사용
- 삭제: `delete` 사용

### Service 구성

- `@Service` 어노테이션 사용
- `domain/{domain}/application` 패키지에 위치
- 트랜잭션 관리 필수 (아래 참조)

## 6. 서비스 계층

### 트랜잭션 관리

- 모든 쓰기 작업에 `@Transactional` 적용
- 읽기 전용 메서드는 `@Transactional(readOnly = true)` 사용
- 트랜잭션 경계는 Service 또는 Facade 메서드 단위로 설정
- 기본 Propagation은 `REQUIRED` 사용

### 의존성 주입

- 생성자 주입 방식 사용 (`@RequiredArgsConstructor` 활용)
- 순환 참조 절대 금지 (Facade 패턴으로 해결)

### 예외 처리

- 비즈니스 로직에서 발생하는 예외는 커스텀 예외 클래스 사용
- `ErrorMessage` Enum을 활용하여 예외 메시지 관리

## 7. DTO와 데이터 매핑

### DTO 설계 원칙

- **모든 DTO는 반드시 `record` 클래스로 작성** (필수)
- Request와 Response DTO 분리
- DTO는 불변(immutable)하며 setter 메서드 절대 금지
- 필드 접근은 record의 자동 생성 accessor 메서드 사용 (예: `id()`)

### record 클래스 사용

```java
// Request DTO 예시
public record LoginRequest(
        @NotBlank String provider,
        @NotBlank String idToken
    ) {

}
```

### DTO 위치

- API 요청/응답 DTO: `api/{domain}/dto/request` 또는 `api/{domain}/dto/response`
- Facade 내부 DTO: `api/{domain}/facade/dto/request` 또는 `api/{domain}/facade/dto/response`
- Domain Service 내부 DTO: `domain/{domain}/application/dto`

### Validation

- Request DTO에 Bean Validation 어노테이션 사용 (`@NotNull`, `@NotBlank` 등)
- Controller 메서드 파라미터에 `@Valid` 적용

## 8. 커스텀 어노테이션

### @Permission

#### 목적

API 엔드포인트에 대한 권한 기반 접근 제어를 수행함.

#### 사용 위치

- Controller 메서드에만 적용 가능 (`@Target(ElementType.METHOD)`)

#### 동작 방식

1. HTTP 요청의 `Authorization` 헤더에서 JWT 토큰 추출
2. 토큰에서 사용자의 Role 정보 파싱
3. 메서드에 지정된 허용 Role과 비교하여 접근 권한 검증
4. 권한이 없으면 `FORBIDDEN` 예외 발생

#### 속성

- `role`: 허용할 Role 배열 (기본값: `Role.ONBOARDING`)
    - `Role.ONBOARDING`: 온보딩 중인 사용자
    - `Role.GENERAL`: 일반 사용자
    - `Role.ADMIN`: 관리자

#### 사용 예시

```java

@Permission(role = {Role.GENERAL})
@GetMapping("/user/info")
public ResponseEntity<BaseResponse<UserInfoResponse>> getUserInfo(@UserId Long userId) {
    // Role.GENERAL 권한을 가진 사용자만 접근 가능
}

@Permission(role = {Role.ONBOARDING, Role.GENERAL})
@PostMapping("/auth/reissue")
public ResponseEntity<BaseResponse<Token>> reissue(@RefreshToken String refreshToken) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [team-confeti/confeti-api-server](https://github.com/team-confeti/confeti-api-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
