---
trigger: always_on
description: providesTags: ["Payment"],
---

# AI Coding Agent Instructions

## Project Overview

Enterprise payment platform using Spring Boot Modulith (backend), React/TypeScript (frontend), and Python tools for development enforcement. Key components:

- `backend/`: Spring Boot modular monolith (Java 21, Spring Boot 3.5)
- `frontend/`: React + Vite + TypeScript web app
- `src/`: Python utilities and enforcement agents
- `tools/`: Constitutional development tools

## Critical Patterns

### Modular Architecture

#### Module Boundaries

- Each module is a self-contained unit in `backend/src/main/java/com/sass/`
- Modules only communicate through well-defined events and interfaces
- Example module structure:
  ```
  backend/payment/
  ├── api/                 # Public interfaces and DTOs
  │   ├── PaymentRequest.java
  │   └── PaymentService.java
  ├── internal/           # Implementation details (private)
  │   ├── StripeAdapter.java
  │   └── PaymentProcessor.java
  ├── events/            # Event definitions
  │   ├── PaymentCompletedEvent.java
  │   └── PaymentFailedEvent.java
  └── PaymentModuleConfiguration.java
  ```

#### Event Communication Examples

1. Payment Processing Flow:

   ```java
   // PaymentService emits event
   @Service
   class PaymentServiceImpl {
       @Autowired
       private ApplicationEventPublisher events;

       @Transactional
       public void processPayment(PaymentRequest request) {
           // Process payment...
           events.publishEvent(new PaymentCompletedEvent(paymentId));
       }
   }

   // Subscription module listens for payment events
   @Service
   class SubscriptionManager {
       @EventListener
       public void onPaymentCompleted(PaymentCompletedEvent event) {
           // Update subscription status...
       }
   }
   ```

2. User Activity Tracking:

   ```java
   // User module emits events
   @Service
   class UserServiceImpl {
       @Autowired
       private ApplicationEventPublisher events;

       public void updateProfile(ProfileUpdateRequest request) {
           // Update profile...
           events.publishEvent(new UserProfileUpdatedEvent(userId));
       }
   }

   // Audit module captures all user events
   @Service
   class AuditLogger {
       @EventListener
       public void logUserActivity(UserProfileUpdatedEvent event) {
           // Log user activity with correlation ID...
       }
   }
   ```

Key Rules:

- Modules never import from other modules' `internal` packages
- Events are immutable and contain only essential data
- Event handlers are idempotent and handle failures gracefully

### Development Workflow

1. Always start infrastructure first:

```bash
docker compose up -d  # Postgres, Redis, Mailhog, Adminer
```

2. Backend development:

```bash
cd backend && ./gradlew bootRun  # Start Spring Boot
./gradlew test jacocoTestReport  # Run tests with coverage
```

3. Frontend development:

```bash
cd frontend
npm install && npm run dev      # Development server
npm test                       # Unit tests
npm run test:e2e              # E2E tests (requires test:e2e:install first)
```

### Test-Driven Development Workflow

#### 1. Write the Test First

```java
// PaymentServiceTest.java
@Test
void shouldDeclinePaymentWhenInsufficientFunds() {
    // Given
    var payment = new PaymentRequest("user123", BigDecimal.valueOf(100));
    when(stripeClient.checkBalance("user123")).thenReturn(BigDecimal.valueOf(50));

    // When
    var result = paymentService.processPayment(payment);

    // Then
    assertThat(result.getStatus()).isEqualTo(PaymentStatus.DECLINED);
    assertThat(result.getReason()).isEqualTo("Insufficient funds");
}
```

#### 2. Run Tests (They Should Fail)

```bash
./gradlew test --tests PaymentServiceTest
```

#### 3. Implement the Feature

```java
@Service
class PaymentService {
    public PaymentResult processPayment(PaymentRequest payment) {
        var balance = stripeClient.checkBalance(payment.getUserId());
        if (balance.compareTo(payment.getAmount()) < 0) {
            return new PaymentResult(PaymentStatus.DECLINED, "Insufficient funds");
        }
        // Process payment...
    }
}
```

#### 4. Verify Tests Pass

```bash
./gradlew test jacocoTestReport
```

#### Frontend Component Testing

1. Unit Testing with Vitest:

```typescript
// PaymentForm.test.tsx
describe('PaymentForm', () => {
  it('should validate card number format', async () => {
    // Given
    const onSubmit = vi.fn();
    render(<PaymentForm onSubmit={onSubmit} />);

    // When
    await userEvent.type(screen.getByLabelText(/card number/i), '4242');
    await userEvent.click(screen.getByRole('button', { name: /submit/i }));

    // Then
    expect(screen.getByText(/invalid card number/i)).toBeInTheDocument();
    expect(onSubmit).not.toHaveBeenCalled();
  });
});
```

2. E2E Testing with Playwright:

```typescript
// payment.spec.ts
test("complete payment flow", async ({ page }) => {
  // Given
  await page.goto("/checkout");
  await page.fill('[data-testid="card-number"]', "4242424242424242");

  // When
  await page.click('button:has-text("Pay Now")');

  // Then
  await expect(page.locator(".payment-success")).toBeVisible();
  await expect(page.locator(".order-id")).toContainText(/\d+/);
});
```

3. Component Mocking Pattern:

```typescript
// PaymentContext.test.tsx
const mockStripeClient = {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lsendel/sass](https://github.com/lsendel/sass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
