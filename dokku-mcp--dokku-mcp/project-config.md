---
trigger: always_on
description: - **One test file per source file**: `application.go` → `application_test.go`
---

# Testing Patterns
## Test Structure and Organization

### File Organization
- **One test file per source file**: `application.go` → `application_test.go`
- **Test packages**: Use `_test` suffix for black-box testing when needed
- **Test helpers**: Create in `testutil` package for reusable test code
- **Integration tests**: Use build tags `//go:build integration`

### Test Function Naming
```go
func TestApplication_Deploy_Success(t *testing.T) // Method_Scenario_ExpectedResult
func TestNewApplication_EmptyName_ReturnsError(t *testing.T)
func TestApplicationRepository_GetByName_NotFound_ReturnsError(t *testing.T)
```

## Go Testing Best Practices

### Table-Driven Tests (Official Pattern)
```go
func TestApplicationName_Validation(t *testing.T) {
    tests := []struct {
        name        string
        input       string
        want        bool
        wantErrMsg  string
    }{
        {
            name:  "valid simple name",
            input: "my-app",
            want:  true,
        },
        {
            name:       "invalid empty name",
            input:      "",
            want:       false,
            wantErrMsg: "name cannot be empty",
        },
        {
            name:       "invalid characters",
            input:      "app$invalid",
            want:       false,
            wantErrMsg: "invalid characters",
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := ValidateApplicationName(tt.input)
            
            if tt.want {
                if err != nil {
                    t.Errorf("expected success, got error: %v", err)
                }
                if !got {
                    t.Errorf("expected true, got false")
                }
            } else {
                if err == nil {
                    t.Errorf("expected error, got success")
                }
                if tt.wantErrMsg != "" && !strings.Contains(err.Error(), tt.wantErrMsg) {
                    t.Errorf("expected error message %q, got %q", tt.wantErrMsg, err.Error())
                }
            }
        })
    }
}
```

### Fuzzing (New Go Feature)
```go
func FuzzApplicationNameValidation(f *testing.F) {
    // Seed corpus with known values
    f.Add("valid-app")
    f.Add("app123")
    f.Add("")
    f.Add("invalid$name")
    
    f.Fuzz(func(t *testing.T, name string) {
        // Should never panic
        defer func() {
            if r := recover(); r != nil {
                t.Errorf("validation panicked with input %q: %v", name, r)
            }
        }()
        
        isValid, err := ValidateApplicationName(name)
        
        // Invariant properties
        if name == "" {
            if isValid {
                t.Errorf("empty name should not be valid")
            }
            if err == nil {
                t.Errorf("empty name should return an error")
            }
        }
        
        // If valid, should be reproducible
        if isValid {
            isValid2, err2 := ValidateApplicationName(name)
            if !isValid2 || err2 != nil {
                t.Errorf("validation inconsistent for %q", name)
            }
        }
    })
}
```

### Benchmarking (Performance Testing)
```go
func BenchmarkApplicationDeploy(b *testing.B) {
    service := setupDeploymentService()
    app := createTestApplication("benchmark-app")
    
    b.ResetTimer()
    b.ReportAllocs() // Report memory allocations
    
    for i := 0; i < b.N; i++ {
        if err := service.Deploy(context.Background(), app); err != nil {
            b.Fatalf("deployment failed: %v", err)
        }
    }
}

func BenchmarkApplicationDeploy_Parallel(b *testing.B) {
    service := setupDeploymentService()
    
    b.RunParallel(func(pb *testing.PB) {
        for pb.Next() {
            app := createTestApplication(fmt.Sprintf("app-%d", rand.Int()))
            if err := service.Deploy(context.Background(), app); err != nil {
                b.Fatalf("parallel deployment failed: %v", err)
            }
        }
    })
}
```

### Mocking with Interfaces
```go
//go:generate mockgen -source=repository.go -destination=mocks/repository_mock.go

func TestApplicationHandler_Deploy(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()
    
    mockRepo := mocks.NewMockApplicationRepository(ctrl)
    mockDeployService := mocks.NewMockDeploymentService(ctrl)
    
    handler := &ApplicationHandler{
        appRepo:       mockRepo,
        deployService: mockDeployService,
    }
    
    // Setup expectations
    mockRepo.EXPECT().
        GetByName(gomock.Any(), "test-app").
        Return(&Application{name: "test-app"}, nil)
    
    mockDeployService.EXPECT().
        Deploy(gomock.Any(), "test-app", gomock.Any()).
        Return(&Deployment{ID: "deploy-123"}, nil)
    
    // Execute test
    result, err := handler.Deploy(context.Background(), "test-app")
    
    // Assertions
    assert.NoError(t, err)
    assert.Equal(t, "deploy-123", result.ID)
}
```

### Integration Tests
```go
//go:build integration

func TestDokkuClient_Integration(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping integration test in short mode")
    }
    
    client := NewDokkuClient("/usr/bin/dokku")
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dokku-MCP/dokku-mcp](https://github.com/dokku-MCP/dokku-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
