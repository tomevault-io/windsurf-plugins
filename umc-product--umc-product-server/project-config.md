---
trigger: always_on
description: 이 문서는 GitHub Copilot이 코드 리뷰 시 참고해야 할 프로젝트별 가이드라인입니다.
---

# UMC Challenger - Copilot Code Review Instructions

## Overview

이 문서는 GitHub Copilot이 코드 리뷰 시 참고해야 할 프로젝트별 가이드라인입니다.  
헥사고날 아키텍처를 따르는 Spring Boot 프로젝트입니다.

---

## Architecture Rules

### Package Structure Validation

```
✅ 올바른 구조:
{domain}/
├── domain/           # Entity, VO, Enum, Domain Service
├── application/
│   ├── port/in/     # UseCase interfaces
│   ├── port/out/    # Repository interfaces
│   └── service/     # UseCase implementations
└── adapter/
    ├── in/          # Controllers, Schedulers
    └── out/         # Persistence, External APIs

❌ 잘못된 구조:
- domain/ 내에 Repository 인터페이스
- adapter/ 내에 비즈니스 로직
- application/service/에 인터페이스 정의
```

### Dependency Direction

```
✅ 허용된 의존 방향:
adapter/in → application/service → application/port
adapter/out → application/port/out
application/service → domain

❌ 금지된 의존:
domain → application (역방향)
domain → adapter (역방향)
application/port → application/service (역방향)
adapter/in → adapter/out (수평 의존)
```

---

## Code Review Checklist

### 1. Entity Review

```java
// ✅ GOOD
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Challenger extends BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Long userId;  // ID 참조만

    @Builder
    private Challenger(...) {
    }

    // 도메인 로직
    public void graduate() {
        validateActive();
        this.status = ChallengerStatus.GRADUATED;
    }
}

// ❌ BAD
@Entity
@Getter
@Setter  // Setter 금지
@NoArgsConstructor  // access level 누락
public class Challenger {
    @ManyToOne
    private Member member;  // 다른 도메인 직접 참조 금지

    // 도메인 로직 없이 getter/setter만
}
```

**체크포인트:**

- [ ] `@Setter` 사용하지 않음
- [ ] `@NoArgsConstructor(access = AccessLevel.PROTECTED)` 사용
- [ ] 다른 도메인 Entity 직접 참조하지 않음 (ID만 저장)
- [ ] 상태 변경은 도메인 메서드로만
- [ ] Builder는 private 또는 package-private
- [ ] `BaseEntity` 상속 (created_at, updated_at)

### 2. UseCase (Port In) Review

```java
// ✅ GOOD
public interface RegisterChallengerUseCase {
    Long register(RegisterChallengerCommand command);
}

public record RegisterChallengerCommand(
        Long userId,
        Long gisuId,
        ChallengerPart part
) {
    // Validation in constructor if needed
    public RegisterChallengerCommand {
        Objects.requireNonNull(userId, "userId must not be null");
    }
}

// ❌ BAD
public interface ChallengerUseCase {  // 너무 포괄적인 이름
    void register(Long userId, Long gisuId, String part);  // primitive 타입 나열

    Challenger getById(Long id);  // Entity 직접 반환
}
```

**체크포인트:**

- [ ] 단일 책임: 하나의 UseCase는 하나의 기능만
- [ ] Command/Query 분리 (CQRS)
- [ ] Command 객체는 record 사용 권장
- [ ] Entity 대신 Info/DTO 반환
- [ ] 네이밍: `{동작}{도메인}UseCase`

### 3. Port (Port Out) Review

```java
// ✅ GOOD
public interface LoadChallengerPort {
    Optional<Challenger> findById(Long id);

    boolean existsByUserIdAndGisuId(Long userId, Long gisuId);
}

public interface SaveChallengerPort {
    Challenger save(Challenger challenger);
}

// ❌ BAD
public interface ChallengerPort {  // Load/Save 분리 안됨
    Challenger findById(Long id);  // Optional 미사용

    void save(Challenger challenger);  // 반환값 없음

    List<ChallengerResponse> findAllWithUserInfo();  // Response DTO 반환
}
```

**체크포인트:**

- [ ] Load/Save Port 분리 (ISP 원칙)
- [ ] `Optional` 적절히 사용
- [ ] Response DTO가 아닌 Entity/Domain 객체 반환
- [ ] 메서드명은 기술 중립적으로

### 4. Service Review

```java
// ✅ GOOD
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class ChallengerQueryService implements GetChallengerUseCase {

    private final LoadChallengerPort loadChallengerPort;

    @Override
    public ChallengerInfo getById(Long challengerId) {
        Challenger challenger = loadChallengerPort.findById(challengerId)
                .orElseThrow(() -> new BusinessException(ErrorCode.CHALLENGER_NOT_FOUND));
        return ChallengerInfo.from(challenger);
    }
}

@Service
@RequiredArgsConstructor
@Transactional
public class ChallengerCommandService implements RegisterChallengerUseCase {
    // Command는 @Transactional (readOnly = false)
}

// ❌ BAD
@Service
public class ChallengerService {  // Command/Query 미분리
    @Autowired  // 필드 주입
    private ChallengerRepository repository;  // Port 대신 Repository 직접 사용

    public ChallengerResponse register(...) {  // Response DTO 반환
        // 검증 로직 없이 바로 저장
        return new ChallengerResponse(repository.save(...));
    }
}
```

**체크포인트:**

- [ ] 생성자 주입 (`@RequiredArgsConstructor`)
- [ ] Command/Query Service 분리
- [ ] Query Service는 `@Transactional(readOnly = true)`
- [ ] Port 인터페이스에 의존 (Repository 직접 사용 X)
- [ ] 적절한 예외 처리
- [ ] Response DTO 대신 Info 객체 반환

### 5. Adapter Review

```java
// ✅ GOOD - Persistence Adapter
@Component
@RequiredArgsConstructor
public class ChallengerPersistenceAdapter
        implements LoadChallengerPort, SaveChallengerPort {

    private final ChallengerRepository repository;

    @Override
    public Optional<Challenger> findById(Long id) {
        return repository.findById(id);
    }
}

// ✅ GOOD - Controller (In Adapter)
@RestController
@RequestMapping("/api/v1/challengers")
@RequiredArgsConstructor
public class ChallengerController {

    private final RegisterChallengerUseCase registerUseCase;

    @PostMapping
    public ApiResponse<Long> register(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UMC-PRODUCT/umc-product-server](https://github.com/UMC-PRODUCT/umc-product-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
