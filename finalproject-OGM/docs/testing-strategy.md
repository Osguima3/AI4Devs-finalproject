# Testing Strategy

## Testing Stack

### Unit Testing
- **Framework**: Deno Test
  - Native testing framework in Deno
  - Excellent TypeScript support
  - Built-in assertions
  - Compatible with Effect TS
  - Fast execution

### Integration Testing
- **Framework**: Deno Test + SuperDeno
  - SuperDeno for HTTP assertions
  - Testing CQRS/ES flows end-to-end
  - Database integration testing
  - Event bus testing

### E2E Testing
- **Framework**: Playwright
  - Cross-browser testing
  - Strong TypeScript support
  - Built-in debugging tools
  - Mobile viewport testing
  - Network interception

### Property Testing
- **Framework**: Fast-Check
  - Property-based testing
  - Excellent TypeScript support
  - Integration with Effect TS
  - Automatic test case generation

## Testing Approach

### Unit Tests
```typescript
// Example of Effect TS unit test
import { Effect, pipe } from "@effect/io/Effect"
import { describe, it } from "std/testing/bdd.ts"
import { assertEquals } from "std/testing/asserts.ts"

describe("TransactionService", () => {
  it("should categorize transaction", () => {
    const program = pipe(
      TransactionService.categorize(mockTransaction),
      Effect.map((category) => category.name)
    )
    
    const result = Effect.runSync(program)
    assertEquals(result, "Groceries")
  })
})
```

### Integration Tests
```typescript
// Example of CQRS flow test
import { superoak } from "superoak"
import { describe, it } from "std/testing/bdd.ts"

describe("Transaction Commands", () => {
  it("should create transaction and update read model", async () => {
    const request = await superoak(app)
    await request
      .post("/api/transactions")
      .send(mockTransaction)
      .expect(201)
    
    // Verify read model was updated
    const readModel = await request
      .get("/api/transactions")
      .expect(200)
    
    assertEquals(readModel.body[0].id, mockTransaction.id)
  })
})
```

### E2E Tests
```typescript
// Example of Playwright test
import { test, expect } from "@playwright/test"

test("user can add transaction", async ({ page }) => {
  await page.goto("/")
  await page.click("[data-test=add-transaction]")
  await page.fill("[data-test=amount]", "42.50")
  await page.fill("[data-test=category]", "Groceries")
  await page.click("[data-test=save]")
  
  await expect(page.locator("[data-test=transaction-list]"))
    .toContainText("42.50")
})
```

## Test Organization

### Directory Structure
```
tests/
├── unit/
│   ├── domain/
│   ├── commands/
│   └── queries/
├── integration/
│   ├── api/
│   ├── events/
│   └── persistence/
├── e2e/
│   ├── flows/
│   └── pages/
└── helpers/
    ├── fixtures/
    └── mocks/
```

### Naming Conventions
- Unit tests: `*.test.ts`
- Integration tests: `*.integration.test.ts`
- E2E tests: `*.spec.ts`
- Test helpers: `*.helper.ts`

## Test Coverage

### Coverage Targets
- Unit Tests: > 80%
- Integration Tests: > 70%
- E2E Tests: Critical paths covered

### Coverage Collection
```typescript
// deno.json
{
  "test": {
    "include": ["tests/"],
    "coverage": {
      "include": ["src/"],
      "exclude": ["src/generated/"]
    }
  }
}
```

## Best Practices

### Test Data Management
- Use factories for test data generation
- Maintain fixtures in version control
- Clean up test data after each test
- Use realistic data shapes

### Mocking Strategy
- Mock external services in integration tests
- Use real implementations in E2E tests
- Prefer dependency injection for easier mocking
- Document mock behaviors

### Performance
- Run tests in parallel when possible
- Use test database for integration tests
- Implement test timeouts
- Monitor test execution times

### Documentation
- Document test setup requirements
- Maintain examples of common test patterns
- Document mocking approaches
- Keep test documentation up to date
