---
trigger: always_on
description: Quick reference and common development tasks for kubernaut
---

# Kubernaut Development Reference

## Quick Start Commands
```bash
# Complete development environment setup
make bootstrap-dev              # Setup everything (except LLM at 192.168.1.169:8080)
make test-integration-dev       # Run integration tests
make cleanup-dev               # Clean up environment

# Testing workflow
make test                      # Unit tests (70%+ coverage, 85-90% confidence)
make test-integration-kind     # Integration tests (20% coverage, 80-85% confidence)
make test-e2e-ocp             # E2E tests (10% coverage, 90-95% confidence)

# Development utilities
make dev-status               # Check environment status
make fmt                      # Format Go code
make lint                     # Run linters
```

## Common Development Patterns

### Adding New AI Provider
1. Implement interface in [pkg/ai/llm/](mdc:pkg/ai/llm/)
2. Add configuration in [config/](mdc:config/) YAML files
3. Update [pkg/ai/holmesgpt/client.go](mdc:pkg/ai/holmesgpt/client.go) for integration
4. Add tests in [test/unit/ai/](mdc:test/unit/ai/) and [test/integration/ai/](mdc:test/integration/ai/)

### Adding New Kubernetes Action
1. Implement in [pkg/platform/executor/executor.go](mdc:pkg/platform/executor/executor.go)
2. Add safety validation logic
3. Update [pkg/workflow/templates/](mdc:pkg/workflow/templates/) if needed
4. Add comprehensive tests covering safety scenarios

### Adding New Business Requirement
1. **MANDATORY**: Follow TDD workflow - write tests first per [00-project-guidelines.mdc](mdc:.cursor/rules/00-project-guidelines.mdc)
2. Map to specific business requirement (BR-[CATEGORY]-[NUMBER] format)
3. Identify test tier: Unit (algorithmic), Integration (cross-component), or E2E (workflow)
4. Add test in appropriate [test/](mdc:test/) subdirectory
5. Use Ginkgo/Gomega BDD framework with clear business requirement naming
6. Update confidence metrics in test documentation

## Key Interfaces to Implement
```go
// Core workflow engine
type WorkflowEngine interface {
    CreateWorkflow(ctx context.Context, alert AlertData) (*Workflow, error)
    ExecuteWorkflow(ctx context.Context, workflow *Workflow) error
}

// AI service integration
type AIProvider interface {
    AnalyzeAlert(ctx context.Context, alert AlertData) (*Analysis, error)
    GenerateWorkflow(ctx context.Context, analysis *Analysis) (*Workflow, error)
}

// Kubernetes operations
type ActionExecutor interface {
    ExecuteAction(ctx context.Context, action Action) error
    ValidateAction(ctx context.Context, action Action) error
}
```

## Environment Variables
```bash
# Core configuration
KUBECONFIG=/path/to/kubeconfig
LOG_LEVEL=info
CONFIG_FILE=config/development.yaml

# AI/ML integration
LLM_ENDPOINT=http://192.168.1.169:8080
LLM_PROVIDER=ollama
LLM_MODEL=hf://ggml-org/gpt-oss-20b-GGUF
HOLMESGPT_ENDPOINT=http://localhost:8090

# Testing configuration
USE_MOCK_LLM=false            # Set to true for CI
USE_FAKE_K8S_CLIENT=false     # Set to true for pure unit tests
SKIP_SLOW_TESTS=false         # Set to true for quick test runs

# Database configuration
DB_HOST=localhost
DB_PORT=5433
DB_NAME=action_history
DB_USER=slm_user
DB_PASSWORD=slm_password_dev
```

## Debugging and Troubleshooting

### Common Issues
1. **AI Service Connection**: Check `LLM_ENDPOINT` and service availability
2. **Database Connection**: Verify PostgreSQL is running on correct port
3. **Kubernetes Access**: Ensure `KUBECONFIG` is set and cluster is accessible
4. **Test Failures**: Check environment setup with `make dev-status`

### Log Analysis
```bash
# Check kubernaut application logs
kubectl logs -f deployment/kubernaut -n prometheus-alerts-slm

# Check HolmesGPT integration logs
kubectl logs -f deployment/holmesgpt -n prometheus-alerts-slm

# Local development logs
tail -f logs/kubernaut.log
```

### Performance Monitoring
- **Metrics Endpoint**: `http://localhost:9090/metrics`
- **Health Check**: `http://localhost:8080/health`
- **AI Response Times**: Monitor via Prometheus metrics
- **Database Performance**: Check connection pool metrics

## Testing Strategies by Component

### AI Components (pkg/ai/**)
- **Unit Tests**: Mock AI responses, test confidence calculation algorithms
- **Integration Tests**: Real AI service calls with known inputs/outputs
- **Performance Tests**: Response time and cost optimization

### Workflow Engine (pkg/workflow/**)
- **Unit Tests**: Workflow step logic, condition evaluation algorithms
- **Integration Tests**: End-to-end workflow execution with real components
- **E2E Tests**: Complete alert-to-resolution scenarios

### Kubernetes Platform (pkg/platform/**)
- **Unit Tests**: Safety validation logic, RBAC permission checks
- **Integration Tests**: Real Kubernetes API operations with Kind clusters
- **E2E Tests**: Multi-cluster scenarios with production-like setup

## Security Considerations
- **RBAC**: Use minimal required permissions for Kubernetes operations
- **Secrets**: Store API keys and credentials in Kubernetes secrets
- **Network**: Implement proper network policies for service isolation
- **Audit**: Log all administrative actions with detailed context
- **Validation**: Sanitize all inputs to prevent injection attacks

## Performance Optimization Tips
- **Caching**: Use Redis for frequent AI responses and embeddings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordigilh/kubernaut](https://github.com/jordigilh/kubernaut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
