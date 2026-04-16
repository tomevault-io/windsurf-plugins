---
trigger: always_on
description: This is a modular monolith Transportation Management System using:
---

# Cursor AI Rules for TMS (Transportation Management System)

## Project Context

This is a modular monolith Transportation Management System using:
- **Domain-Driven Design (DDD)** with aggregates and value objects
- **Hexagonal Architecture** (Ports & Adapters) with strict layer separation
- **CQRS** (Command Query Responsibility Segregation) with read/write database separation
- **Event-Driven Architecture** with RabbitMQ and outbox pattern
- **Spring Modulith** for module boundary enforcement

**Tech Stack:** Java 21, Spring Boot 3.x, PostgreSQL, RabbitMQ, Maven

---

## Critical Architecture Rules

### Layer Separation (STRICT ENFORCEMENT)

#### Domain Layer (`domain/`)
- ✅ **ONLY pure Java** - NO framework dependencies
- ✅ Contains: Aggregates, Value Objects, Domain Events, Domain Exceptions
- ✅ Business logic ONLY lives here
- ❌ **NEVER** import Spring, JPA, Jackson, or any framework
- ❌ **NEVER** import infrastructure or application packages

**Example:**
```java
// ✅ CORRECT - Pure domain
public class Company extends AbstractAggregateRoot {
    private final CompanyId companyId;
    private final Cnpj cnpj;
    // ... pure business logic
}

// ❌ WRONG - Framework dependency
@Entity
public class Company extends AbstractAggregateRoot { ... }
```

#### Application Layer (`application/`)
- ✅ Contains: Use Cases, Repository Interfaces, Presenters (interfaces only)
- ✅ Depends ONLY on `domain` package
- ✅ Orchestrates business operations
- ❌ **NEVER** import infrastructure packages
- ❌ **NEVER** contain HTTP, database, or messaging logic

#### Infrastructure Layer (`infrastructure/`)
- ✅ Contains: REST Controllers, JPA Entities, Repository Implementations, DTOs, Messaging
- ✅ Implements interfaces from `application` layer
- ✅ All Spring annotations live here
- ✅ Can depend on `domain` and `application`

---

## Mandatory Patterns

### 1. Use Case Pattern (ALWAYS FOLLOW)

**Write Use Case:**
```java
@DomainService
@Cqrs(DatabaseRole.WRITE)
public class CreateCompanyUseCase implements UseCase<CreateCompanyUseCase.Input, CreateCompanyUseCase.Output> {

    private final CompanyRepository companyRepository;

    public CreateCompanyUseCase(CompanyRepository companyRepository) {
        this.companyRepository = companyRepository;
    }

    @Override
    public Output execute(Input input) {
        // 1. Validation
        if (companyRepository.getCompanyByCnpj(new Cnpj(input.cnpj())).isPresent()) {
            throw new ValidationException("Company already exists");
        }

        // 2. Domain logic (aggregate creates itself)
        Company company = Company.createCompany(input.name(), input.cnpj(), input.types(), input.configuration());

        // 3. Persistence
        company = companyRepository.create(company);

        // 4. Return output
        return new Output(company.getCompanyId().value(), company.getName());
    }

    public record Input(String name, String cnpj, Set<CompanyType> types, Map<String, Object> configuration) {}
    public record Output(UUID companyId, String name) {}
}
```

**Read Use Case:**
```java
@DomainService
@Cqrs(DatabaseRole.READ)
public class GetCompanyByIdUseCase implements UseCase<GetCompanyByIdUseCase.Input, GetCompanyByIdUseCase.Output> {

    private final CompanyRepository companyRepository;

    public GetCompanyByIdUseCase(CompanyRepository companyRepository) {
        this.companyRepository = companyRepository;
    }

    @Override
    public Output execute(Input input) {
        Company company = companyRepository.getCompanyById(new CompanyId(input.companyId()))
                .orElseThrow(() -> new NotFoundException("Company not found"));

        return new Output(company.getCompanyId().value(), company.getName(), company.getCnpj().value());
    }

    public record Input(UUID companyId) {}
    public record Output(UUID companyId, String name, String cnpj) {}
}
```

**Key Rules:**
- ✅ One use case per operation (Single Responsibility)
- ✅ Always annotate with `@DomainService` and `@Cqrs(DatabaseRole.WRITE or READ)`
- ✅ Input/Output as nested records
- ✅ Constructor injection only
- ✅ Business validation in use case, domain invariants in aggregate

---

### 2. REST Controller Pattern

```java
@RestController
@RequestMapping("companies")
@Cqrs(DatabaseRole.WRITE)
public class CreateController {

    private final CreateCompanyUseCase createCompanyUseCase;
    private final DefaultRestPresenter defaultRestPresenter;
    private final RestUseCaseExecutor restUseCaseExecutor;

    public CreateController(CreateCompanyUseCase createCompanyUseCase,
                           DefaultRestPresenter defaultRestPresenter,
                           RestUseCaseExecutor restUseCaseExecutor) {
        this.createCompanyUseCase = createCompanyUseCase;
        this.defaultRestPresenter = defaultRestPresenter;
        this.restUseCaseExecutor = restUseCaseExecutor;
    }

    @PostMapping
    public Object create(@RequestBody CreateCompanyDTO createCompanyDTO) {
        return restUseCaseExecutor
                .from(createCompanyUseCase)
                .withInput(createCompanyDTO)
                .mapOutputTo(CreateCompanyResponseDTO.class)
                .presentWith(output -> defaultRestPresenter.present(output, HttpStatus.CREATED.value()))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lmmoreira) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
