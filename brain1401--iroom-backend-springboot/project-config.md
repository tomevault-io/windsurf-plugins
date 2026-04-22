---
trigger: always_on
description: REST API 설계 및 응답 표준화 규칙
---


# API 설계 및 응답 표준화

## 🎯 API 설계 원칙

### RESTful API 설계
- **리소스 중심** URL 설계
- **HTTP 메서드** 의미에 맞는 사용
- **일관된 응답 형식** (ApiResponse 래퍼)
- **적절한 상태 코드** 사용

### API 문서화
- **OpenAPI 3.0** (SpringDoc) 활용
- **@Tag**, **@Operation** 상세 작성
- **@Schema** 어노테이션으로 DTO 문서화

## 📋 표준 응답 형식 (ApiResponse)

### ✅ 기본 ApiResponse 구조
```java
/**
 * API 표준 응답 래퍼
 */
public record ApiResponse<T>(
    @Schema(description = "처리 결과", example = "SUCCESS") 
    String result,
    
    @Schema(description = "응답 메시지", example = "요청이 성공적으로 처리되었습니다") 
    String message,
    
    @Schema(description = "응답 데이터") 
    T data
) {
    public ApiResponse {
        // 불변성: result, message는 null이 될 수 없음
        if (result == null || result.isBlank()) 
            throw new IllegalArgumentException("result는 필수입니다");
        if (message == null) 
            throw new IllegalArgumentException("message는 필수입니다");
    }
    
    /**
     * 성공 응답 생성 (데이터 포함)
     */
    public static <T> ApiResponse<T> success(String message, T data) {
        return new ApiResponse<>("SUCCESS", message, data);
    }
    
    /**
     * 성공 응답 생성 (데이터 없음)
     */
    public static ApiResponse<Void> success(String message) {
        return new ApiResponse<>("SUCCESS", message, null);
    }
    
    /**
     * 오류 응답 생성
     */
    public static ApiResponse<Void> error(String message) {
        return new ApiResponse<>("ERROR", message, null);
    }
}
```

### ✅ Controller에서 ApiResponse 활용
```java
/**
 * 사용자 관리 컨트롤러
 */
@RestController
@RequestMapping("/users")
@RequiredArgsConstructor
@Tag(name = "사용자 API", description = "학생 로그인 및 정보 관리 API")
@Validated
public class UserController {
    
    private final UserService userService;
    
    /**
     * 사용자 로그인
     */
    @PostMapping("/login")
    @Operation(
        summary = "사용자 로그인",
        description = "학생이 이름과 전화번호로 로그인합니다",
        responses = {
            @ApiResponse(responseCode = "200", description = "로그인 성공"),
            @ApiResponse(responseCode = "400", description = "입력 데이터 검증 실패"),
            @ApiResponse(responseCode = "404", description = "존재하지 않는 사용자")
        }
    )
    public ApiResponse<UserLoginResponse> login(
            @Parameter(description = "로그인 요청 정보") 
            @Valid @RequestBody UserLoginRequest request) {
        
        UserLoginResponse response = userService.login(request);
        return ApiResponse.success("로그인에 성공했습니다", response);
    }
    
    /**
     * 사용자 정보 조회
     */
    @GetMapping("/{id}")
    @Operation(summary = "사용자 정보 조회", description = "사용자 ID로 상세 정보를 조회합니다")
    public ApiResponse<UserDto> getUser(
            @Parameter(description = "사용자 ID", example = "1") 
            @PathVariable Long id) {
        
        UserDto user = userService.findById(id);
        return ApiResponse.success("사용자 정보를 조회했습니다", user);
    }
    
    /**
     * 사용자 목록 조회 (페이징)
     */
    @GetMapping
    @Operation(summary = "사용자 목록 조회", description = "페이징을 지원하는 사용자 목록을 조회합니다")
    public ApiResponse<Page<UserDto>> getUsers(
            @Parameter(description = "페이지 번호 (0부터 시작)", example = "0")
            @RequestParam(defaultValue = "0") int page,
            
            @Parameter(description = "페이지 크기", example = "20")
            @RequestParam(defaultValue = "20") int size,
            
            @Parameter(description = "정렬 기준", example = "name")
            @RequestParam(defaultValue = "id") String sort,
            
            @Parameter(description = "정렬 방향", example = "asc")
            @RequestParam(defaultValue = "asc") String direction) {
        
        Sort sortOrder = Sort.by(Sort.Direction.fromString(direction), sort);
        Pageable pageable = PageRequest.of(page, size, sortOrder);
        
        Page<UserDto> users = userService.findAll(pageable);
        return ApiResponse.success("사용자 목록을 조회했습니다", users);
    }
}
```

## 🔢 HTTP 상태 코드 활용

### 성공 상태 코드
| 코드               | 설명            | 사용 시점            | 예시              |
| ------------------ | --------------- | -------------------- | ----------------- |
| **200 OK**         | 요청 성공       | GET, PUT, PATCH 성공 | 조회, 수정 완료   |
| **201 Created**    | 생성 성공       | POST 성공            | 사용자, 시험 생성 |
| **204 No Content** | 성공, 응답 없음 | DELETE 성공          | 삭제 완료         |

### ✅ 상태 코드 적용 예시
```java
@RestController
@RequestMapping("/exams")
@RequiredArgsConstructor
@Tag(name = "시험 관리 API")
public class ExamController {
    
    private final ExamService examService;
    
    /**
     * 시험 생성 - 201 Created
     */
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    @Operation(summary = "시험 생성", description = "새로운 시험을 등록합니다")
    public ApiResponse<ExamCreateResponse> createExam(
            @Valid @RequestBody ExamCreateRequest request) {
        
        ExamCreateResponse exam = examService.createExam(request);
        return ApiResponse.success("시험이 성공적으로 등록되었습니다", exam);
    }
    
    /**
     * 시험 조회 - 200 OK
     */
    @GetMapping("/{id}")
    @Operation(summary = "시험 상세 조회")
    public ApiResponse<ExamDetailResponse> getExam(@PathVariable Long id) {
        ExamDetailResponse exam = examService.getExamDetail(id);
        return ApiResponse.success("시험 정보를 조회했습니다", exam);
    }
    
    /**
     * 시험 수정 - 200 OK
     */
    @PutMapping("/{id}")
    @Operation(summary = "시험 정보 수정")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brain1401) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
