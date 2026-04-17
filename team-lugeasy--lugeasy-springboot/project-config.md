---
trigger: always_on
description: - **프로젝트명**: Lugeasy (러기지 관리 서비스)
---

# Lugeasy SpringBoot 프로젝트 커서 규칙

## 프로젝트 개요
- **프로젝트명**: Lugeasy (러기지 관리 서비스)
- **기술 스택**: Spring Boot 3.2.2, Java 21, MariaDB, Redis, JWT
- **아키텍처**: 계층형 아키텍처 (Controller → Service → Repository)
- **패키지 구조**: 도메인별 패키지 분리 (user, auth, review, match, common)

## 코딩 컨벤션

### 1. 패키지 구조
```
com.jimjim.lugeasy/
├── common/           # 공통 모듈
│   ├── config/       # 설정 클래스 (SecurityConfig, SwaggerConfig, RedisConfig)
│   ├── entity/       # 공통 엔티티 (BaseEntity)
│   ├── error/        # 에러 처리 (ControllerAdvice, ErrorCode, RestApiException)
│   ├── response/     # 공통 응답 형식 (BaseResponse)
│   └── security/     # 보안 관련 (JWT, AuthenticationMember, CustomUserDetails)
├── {domain}/         # 도메인별 패키지 (user, auth, review, match)
│   ├── application/  # 애플리케이션 서비스 (Facade 패턴)
│   │   └── v1/       # API 버전별 분리
│   │       ├── dto/  # Request/Response DTO
│   │       ├── {Domain}Facade.java      # Facade 인터페이스
│   │       └── {Domain}FacadeImpl.java  # Facade 구현체
│   ├── domain/       # 도메인 엔티티 (Entity, Enum, Value Object)
│   ├── errorCode/    # 도메인별 에러 코드 (DomainErrorCode)
│   ├── infrastructure/ # 리포지토리 (Repository 인터페이스)
│   ├── representation/ # 컨트롤러 (REST API 엔드포인트)
│   │   └── v1/       # API 버전별 분리
│   │       ├── dto/  # 컨트롤러 전용 DTO
│   │       └── {Domain}Controller.java
│   └── service/      # 도메인 서비스 (비즈니스 로직)
│       ├── {Domain}Service.java        # 서비스 인터페이스
│       └── impl/
│           └── {Domain}ServiceImpl.java # 서비스 구현체
```

#### 패키지 구조 설명:
- **application/**: Facade 패턴으로 여러 도메인을 조합하는 애플리케이션 서비스
- **domain/**: 순수한 도메인 로직과 엔티티
- **infrastructure/**: 데이터 접근 계층 (Repository)
- **representation/**: 외부 인터페이스 (REST API)
- **service/**: 도메인별 비즈니스 로직
- **errorCode/**: 도메인별 예외 처리

#### 실제 도메인별 예시:
```
com.jimjim.lugeasy.review/
├── application/v1/
│   ├── dto/
│   │   ├── ReviewRequestDTO.java
│   │   └── ReviewResponseDTO.java
│   ├── ReviewFacade.java
│   └── ReviewFacadeImpl.java
├── domain/
│   └── Review.java
├── errorCode/
│   └── ReviewErrorCode.java
├── infrastructure/
│   └── ReviewRepository.java
├── representation/v1/
│   ├── dto/
│   │   └── ReviewControllerDTO.java
│   └── ReviewController.java
└── service/
    ├── ReviewService.java
    └── impl/
        └── ReviewServiceImpl.java
```

#### 계층 간 의존성 규칙:
1. **Controller** → **Facade** → **Service** → **Repository**
2. **Facade**는 여러 도메인의 **Service**를 조합
3. **Service**는 단일 도메인의 비즈니스 로직만 처리
4. **Repository**는 데이터 접근만 담당

### 2. 네이밍 컨벤션
- **클래스명**: PascalCase (예: `HostController`, `MemberService`)
- **메서드명**: camelCase (예: `getAuthenticatedHostList()`, `updateMember()`)
- **변수명**: camelCase (예: `hostId`, `memberRepository`)
- **상수명**: UPPER_SNAKE_CASE (예: `JWT_SECRET`)
- **패키지명**: 소문자 (예: `com.jimjim.lugeasy.user.domain`)

### 3. 어노테이션 사용 규칙
```java
// 엔티티
@Entity
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Member extends BaseEntity {
    // ...
}

// 서비스
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true) // 조회용
@Transactional // 수정용
public class HostServiceImpl implements HostService {
    // ...
}

// 컨트롤러
@RestController
@RequestMapping("/api/v1/hosts")
@RequiredArgsConstructor
@Tag(name = "Host", description = "호스트 관련 API")
public class HostController {
    // ...
}
```

### 4. 에러 처리 패턴
```java
// 에러 코드 정의
@Getter
@AllArgsConstructor
public enum MemberErrorCode implements ErrorCodeInterface {
    MEMBER_NOT_FOUND("MEMBER001", "Member가 존재하지 않습니다.", HttpStatus.NOT_FOUND);
    
    @Override
    public ErrorCode getErrorCode() {
        return ErrorCode.builder()
                .code(code)
                .message(message)
                .httpStatus(httpStatus)
                .build();
    }
}

// 예외 발생
throw new RestApiException(MemberErrorCode.MEMBER_NOT_FOUND);
```

### 5. API 응답 형식
```java
// 성공 응답
return BaseResponse.onSuccess(result);

// 실패 응답 (ControllerAdvice에서 자동 처리)
throw new RestApiException(ErrorCode);
```

### 6. JPA 엔티티 규칙
```java
@Entity
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Entity extends BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private String name;
    
    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id", nullable = false)
    private Member member;
    
    @OneToMany(mappedBy = "host", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<HostTime> availableTimes = new ArrayList<>();
}
```

### 7. DTO 패턴
```java
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Schema(description = "호스트 상세 정보 DTO")
public class HostDetailResponseDTO {
    @Schema(description = "호스트 ID")
    private Long id;
    
    @Schema(description = "호스트 이름")
    private String name;
}
```

### 8. Swagger 문서화
```java
@Operation(summary = "호스트 목록 조회", description = "인증된 호스트들의 목록을 조회합니다.")
@ApiResponses(value = {
    @ApiResponse(responseCode = "200", description = "호스트 목록 조회 성공",
        content = @Content(mediaType = "application/json",
            examples = @ExampleObject(
                name = "호스트 목록 조회 성공 응답",
                value = """
                {
                  "timestamp": "2025-08-31T09:45:40.426159645Z",
                  "code": "COMMON200",
                  "message": "요청에 성공하였습니다.",
                  "result": [...]
                }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Team-Lugeasy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
