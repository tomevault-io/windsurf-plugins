---
trigger: always_on
description: This file provides guidance when working with code in this repository.
---

# AI TOOL GUIDANCE

This file provides guidance when working with code in this repository.

## Technology Stack

This is a Vaadin application built with:
- Kotlin, Java 21
- Spring Boot, Vaadin
- Spring Data JPA + MySQL (H2 for tests)
- Gradle build system
- KFmt (Google style), Jacoco, SonarQube

## Coding Conventions

- Entities use Kotlin properties (`var`/`val`) with `private set` for computed/readonly fields.
- Constructor injection throughout (no `@Autowired` on fields).
- **Always format code with KFmt** (`./gradlew ktfmtFormat`) before committing. The project uses Google style. CI will fail on unformatted code.
- **KDoc**: Every public class, method, field, and enum value that is not trivially self-explained must have a concise KDoc comment explaining the business logic. Keep KDoc short and focused on the "why", not the "what".

## Development Commands

### Running the Application
```bash
./gradlew bootRun                # Start in development mode
```

The application will be available at http://localhost:8080

### Formatting
```bash
./gradlew ktfmtFormat                            # Format all Kotlin files (Google style)
```

### Building for Production
```bash
./gradlew bootJar -Pvaadin.productionMode=true   # Build production JAR
docker build -t command-manager:latest .          # Build Docker image
```

### Testing
```bash
./gradlew test                                                        # Run all tests
./gradlew test --tests "fr.axl.lvy.order.OrderCodigServiceTest"           # Run a single test class
./gradlew test --tests "fr.axl.lvy.order.OrderCodigServiceTest.someTest"  # Run a single test method
```

## Architecture

This project follows a **feature-based package structure** rather than traditional layered architecture. Code is organized by functional units (features), not by technical layers.

### Package Structure

- **`fr.axl.lvy.base`**: Reusable components and base classes
  - `BaseEntity.kt`: MappedSuperclass with id, createdAt, updatedAt, equals/hashCode
  - `SoftDeletableEntity.kt`: Extends BaseEntity, adds deletedAt with softDelete()/restore()
  - `ui.MainLayout`: AppLayout with SideNav, auto-populated from `@Menu` annotations
  - `ui.ViewToolbar`: Reusable toolbar component for views

- **`fr.axl.lvy.client`**: Client/supplier management
  - `Client.kt`: Entity (COMPANY/INDIVIDUAL, roles: CLIENT/PRODUCER/BOTH, multi-company visibility)
  - `contact/Contact.kt`: Contact person entity (roles: PRIMARY/BILLING/TECHNICAL/OTHER)
  - `ClientRepository.kt`, `ContactRepository.kt`, `ClientService.kt`
  - `ui/`: ClientListView, ClientFormDialog, ContactFormDialog

- **`fr.axl.lvy.product`**: Product catalog
  - `Product.kt`: Entity (types: PRODUCT/SERVICE, MTO flag, selling/purchase prices)
  - `ProductRepository.kt`, `ProductService.kt`
  - `ui/`: ProductListView, ProductFormDialog

- **`fr.axl.lvy.quote`**: Quotations/estimates
  - `Quote.kt`: Entity with status workflow (DRAFT → SENT → ACCEPTED/REFUSED/EXPIRED)
  - `QuoteRepository.kt`, `QuoteService.kt` (includes `convertToOrderCodig()`)
  - `ui/`: QuoteListView, QuoteFormDialog

- **`fr.axl.lvy.order`**: Order management
  - `OrderCodig.kt`: Customer order (DRAFT → CONFIRMED → DELIVERED → INVOICED or CANCELLED)
  - `OrderNetstone.kt`: Supplier/MTO order (SENT → CONFIRMED → RECEIVED or CANCELLED)
  - Services with state machines, duplicate(), handleMto()
  - `ui/`: CommandCodigListView, CommandCodigFormDialog, CommandNetstoneListView, CommandNetstoneFormDialog

- **`fr.axl.lvy.documentline`**: Reusable line items shared across documents
  - `DocumentLine.kt`: Linked via DocumentType enum + documentId, with recalculate()
  - `DocumentLineRepository.kt`
  - `ui/DocumentLineEditor.kt`: Inline line item editor component

- **`fr.axl.lvy.delivery`**: Delivery notes
  - `DeliveryNoteCodig.kt`: Customer delivery (PREPARED → SHIPPED → DELIVERED → INCIDENT)
  - `DeliveryNoteNetstone.kt`: Supplier delivery (IN_TRANSIT → ARRIVED → INSPECTED → INCIDENT)

- **`fr.axl.lvy.invoice`**: Invoices
  - `InvoiceCodig.kt`: Customer invoice (DRAFT → ISSUED → OVERDUE → PAID → CANCELLED/CREDIT_NOTE)
  - `InvoiceNetstone.kt`: Supplier invoice (RECEIVED → VERIFIED → PAID → DISPUTED)

- **`fr.axl.lvy.user`**: System users
  - `User.kt`: Entity (roles: ADMIN/COLLABORATOR/ACCOUNTANT, multi-company: CODIG/NETSTONE/BOTH, canSee() visibility logic)

- **`Application.kt`**: Main entry point, `@SpringBootApplication` and `@Theme("default")`

### Key Architecture Patterns

1. **Feature Packages**: Each feature is self-contained with its own entity, repository, service, UI, and tests
2. **Entity Hierarchy**: BaseEntity (id, audit fields) → SoftDeletableEntity (adds soft delete) → TotalizableDocument (totals, incoterms, notes) → CodigDocument (client, addresses, currency) → OrderCodig/SalesCodig
3. **Navigation**: Views use `@Route` and `@Menu` annotations. MainLayout builds navigation automatically
4. **Service Layer**: `@Transactional` for writes, `@Transactional(readOnly = true)` for reads
5. **Soft Delete**: Most entities extend SoftDeletableEntity with `softDelete()` and `restore()` methods
6. **Audit Fields**: `createdAt`/`updatedAt` set via `@PrePersist`/`@PreUpdate` in BaseEntity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Axl-Lvy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
