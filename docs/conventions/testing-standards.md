# Testing Standards

This document defines testing requirements, conventions, and best practices for the Ultrawork project.

## Testing Philosophy

- **Test behavior, not implementation**: Focus on what the code does, not how it does it
- **Write tests first when fixing bugs**: Reproduce the bug, then fix it
- **Maintain test quality**: Tests should be as clean as production code
- **Fast feedback**: Unit tests should run in milliseconds, integration tests in seconds

## Backend Testing (Kotlin/Spring Boot)

### Technology Stack

- **Framework**: JUnit 5 (Jupiter)
- **Mocking**: MockK 1.13.10
- **Integration**: Testcontainers 1.19.7
- **Database**: H2 (in-memory), PostgreSQL (containers)

### Test Structure

**Location**: `backend/src/test/kotlin/com/ultrawork/`

**Package Structure**:
```
com.ultrawork/
├── api/
│   ├── controller/     # Controller tests
│   └── advice/         # Exception handler tests
├── domain/
│   └── service/        # Service tests (unit & integration)
└── infrastructure/
    ├── redis/          # Redis integration tests
    └── health/         # Health indicator tests
```

### Test Types

#### Unit Tests

**Purpose**: Test individual components in isolation

**Naming Convention**: `<ClassName>Test.kt`

**Example**: `JwtBlacklistServiceTest.kt`

```kotlin
class JwtBlacklistServiceTest {
    private val redisTemplate: RedisTemplate<String, String> = mockk()
    private val valueOps: ValueOperations<String, String> = mockk()
    private val jwtBlacklistService = JwtBlacklistService(redisTemplate)

    @Test
    fun `should add token to blacklist`() {
        val token = "test-jwt-token-123"
        val expiry = Duration.ofHours(24)

        every { redisTemplate.opsForValue() } returns valueOps
        every { valueOps.set("jwt:blacklist:$token", "revoked", expiry) } returns Unit

        jwtBlacklistService.addToken(token, expiry)

        verify { valueOps.set("jwt:blacklist:$token", "revoked", expiry) }
    }
}
```

**Characteristics**:
- Use MockK for dependencies
- Test one behavior per test method
- Use backtick test names for readability
- Follow Arrange-Act-Assert pattern

#### Integration Tests

**Purpose**: Test components with real dependencies

**Naming Convention**: `<ClassName>IntegrationTest.kt`

**Example**: `SystemMetricsIntegrationTest.kt`

**Characteristics**:
- Use `@SpringBootTest` for full context
- Use Testcontainers for databases
- Test actual database queries
- Verify end-to-end behavior

#### Controller Tests

**Purpose**: Test HTTP endpoints and request/response handling

**Naming Convention**: `<ControllerName>ControllerTest.kt`

**Example**: `HealthControllerTest.kt`

**Characteristics**:
- Use `@WebMvcTest` for focused testing
- Mock service layer
- Test request validation
- Verify response status and body
- Test error handling

### Test Naming

Use descriptive backtick names that read like specifications:

**Good**:
```kotlin
@Test
fun `should add token to blacklist`()

@Test
fun `should return false for non-blacklisted token`()

@Test
fun `should throw exception when token is invalid`()
```

**Bad**:
```kotlin
@Test
fun testAddToken()

@Test
fun test1()
```

### Assertions

Use JUnit 5 assertions:

```kotlin
import org.junit.jupiter.api.Assertions.*

assertTrue(condition)
assertFalse(condition)
assertEquals(expected, actual)
assertNotNull(value)
assertThrows<ExceptionType> { code() }
```

### Mocking with MockK

**Basic mocking**:
```kotlin
private val repository: UserRepository = mockk()

every { repository.findById(userId) } returns user
verify { repository.save(any()) }
```

**Relaxed mocks** (return default values):
```kotlin
private val service: UserService = mockk(relaxed = true)
```

**Argument matchers**:
```kotlin
every { service.create(any()) } returns user
every { service.update(match { it.id > 0 }) } returns user
```

### Test Data

**Use meaningful test data**:
```kotlin
val testUser = User(
    id = 1L,
    email = "test@example.com",
    name = "Test User"
)
```

**Avoid magic values**:
```kotlin
// Bad
val user = createUser(1, "test", "pass")

// Good
val userId = 1L
val email = "test@example.com"
val user = createUser(userId, email, password)
```

### Current Test Coverage

**Total Tests**: 8 test files

**Coverage by Layer**:
- Controllers: 2 tests (Health, SystemMetrics)
- Services: 2 tests (JwtBlacklist, SystemMetrics)
- Infrastructure: 2 tests (Redis, Health Indicator)
- Exception Handling: 1 test (GlobalExceptionHandler)
- Integration: 1 test (SystemMetrics)

### Running Tests

```bash
cd backend

# Run all tests
./gradlew test

# Run specific test class
./gradlew test --tests JwtBlacklistServiceTest

# Run with coverage
./gradlew test jacocoTestReport

# Run integration tests only
./gradlew test --tests "*IntegrationTest"
```

## Frontend Testing (TypeScript/Next.js)

### Technology Stack

- **Framework**: Vitest 4.0.18
- **React Testing**: Testing Library 16.3.2
- **E2E**: Playwright 1.58.2
- **Mocking**: Vitest built-in mocks

### Test Structure

**Location**: `frontend/src/__tests__/`

**File Naming**: `<component>.test.tsx` or `<module>.test.ts`

### Test Types

#### Component Tests

**Purpose**: Test React components in isolation

**Example**: `layout.test.tsx`

```typescript
import { render, screen } from "@testing-library/react";
import { describe, it, expect, vi } from "vitest";

vi.mock("next/font/google", () => ({
  Geist: () => ({ variable: "--font-geist-sans" }),
  Geist_Mono: () => ({ variable: "--font-geist-mono" }),
}));

describe("RootLayout", () => {
  it("renders children within html and body tags", async () => {
    const { default: RootLayout } = await import("@/app/layout");

    const { container } = render(
      <RootLayout>
        <div data-testid="child">Ultrawork</div>
      </RootLayout>,
      {
        container: document.documentElement,
      }
    );

    expect(screen.getByTestId("child")).toBeInTheDocument();
    expect(screen.getByText("Ultrawork")).toBeInTheDocument();

    const body = container.querySelector("body");
    expect(body).toHaveClass("antialiased");
  });
});
```

**Characteristics**:
- Use Testing Library queries
- Test user-visible behavior
- Mock external dependencies
- Use `data-testid` for reliable selection

#### Hook Tests

**Purpose**: Test custom React hooks

```typescript
import { renderHook, act } from "@testing-library/react";
import { describe, it, expect } from "vitest";
import { useCounter } from "@/hooks/useCounter";

describe("useCounter", () => {
  it("increments counter", () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });
});
```

#### Utility Tests

**Purpose**: Test pure functions and utilities

```typescript
import { describe, it, expect } from "vitest";
import { formatDate } from "@/lib/utils";

describe("formatDate", () => {
  it("formats date correctly", () => {
    const date = new Date("2024-01-15");
    expect(formatDate(date)).toBe("January 15, 2024");
  });
});
```

#### E2E Tests (Playwright)

**Purpose**: Test complete user workflows

**Location**: `frontend/e2e/`

```typescript
import { test, expect } from "@playwright/test";

test("user can log in", async ({ page }) => {
  await page.goto("/login");
  
  await page.fill('[name="email"]', "user@example.com");
  await page.fill('[name="password"]', "password123");
  await page.click('button[type="submit"]');
  
  await expect(page).toHaveURL("/dashboard");
  await expect(page.locator("h1")).toContainText("Dashboard");
});
```

### Testing Library Queries

**Priority order** (most to least preferred):

1. **Accessible queries** (preferred):
   - `getByRole`: `getByRole('button', { name: /submit/i })`
   - `getByLabelText`: `getByLabelText('Email')`
   - `getByPlaceholderText`: `getByPlaceholderText('Enter email')`
   - `getByText`: `getByText('Welcome')`

2. **Semantic queries**:
   - `getByAltText`: `getByAltText('Profile picture')`
   - `getByTitle`: `getByTitle('Close')`

3. **Test IDs** (last resort):
   - `getByTestId`: `getByTestId('submit-button')`

### Query Variants

- `getBy*`: Throws if not found (use for assertions)
- `queryBy*`: Returns null if not found (use for non-existence checks)
- `findBy*`: Async, waits for element (use for async rendering)

```typescript
// Element must exist
expect(screen.getByText("Hello")).toBeInTheDocument();

// Element should not exist
expect(screen.queryByText("Error")).not.toBeInTheDocument();

// Wait for async element
const message = await screen.findByText("Success");
```

### User Interactions

Use `@testing-library/user-event` for realistic interactions:

```typescript
import userEvent from "@testing-library/user-event";

it("handles form submission", async () => {
  const user = userEvent.setup();
  render(<LoginForm />);

  await user.type(screen.getByLabelText("Email"), "test@example.com");
  await user.type(screen.getByLabelText("Password"), "password123");
  await user.click(screen.getByRole("button", { name: /submit/i }));

  expect(await screen.findByText("Welcome")).toBeInTheDocument();
});
```

### Mocking

**Mock modules**:
```typescript
vi.mock("@/lib/api", () => ({
  fetchUser: vi.fn(() => Promise.resolve({ id: 1, name: "Test" })),
}));
```

**Mock Next.js router**:
```typescript
vi.mock("next/navigation", () => ({
  useRouter: () => ({
    push: vi.fn(),
    pathname: "/",
  }),
}));
```

**Mock environment variables**:
```typescript
vi.stubEnv("NEXT_PUBLIC_API_URL", "http://localhost:3000");
```

### Running Tests

```bash
cd frontend

# Run all tests
npm run test

# Watch mode
npm run test:watch

# E2E tests
npm run test:e2e

# E2E with UI
npx playwright test --ui
```

## Test Coverage Goals

### Current Status

- **Backend**: 8 test files covering core services and controllers
- **Frontend**: 1 test file (layout component)

### Target Coverage

**Backend**:
- Controllers: 80%+ coverage
- Services: 90%+ coverage
- Utilities: 95%+ coverage
- Integration: Key workflows covered

**Frontend**:
- Components: 80%+ coverage
- Hooks: 90%+ coverage
- Utilities: 95%+ coverage
- E2E: Critical user paths covered

### Coverage Priorities

1. **Business logic**: Services, domain models
2. **API endpoints**: Controllers, request/response handling
3. **Error handling**: Exception handlers, validation
4. **User interactions**: Forms, navigation, state changes
5. **Edge cases**: Boundary conditions, error states

## Best Practices

### General

- **Test behavior, not implementation**: Don't test private methods
- **One assertion per test**: Focus on single behavior
- **Independent tests**: Tests should not depend on each other
- **Fast tests**: Keep unit tests under 100ms
- **Descriptive names**: Test name should describe expected behavior

### Backend

- **Use MockK for mocking**: Prefer `mockk()` over manual mocks
- **Test exceptions**: Verify error handling with `assertThrows`
- **Use Testcontainers**: Real databases for integration tests
- **Clean up resources**: Use `@AfterEach` for cleanup
- **Avoid `@SpringBootTest` for unit tests**: Too slow, use focused slices

### Frontend

- **Query by accessibility**: Use `getByRole`, `getByLabelText`
- **Avoid implementation details**: Don't test state or props directly
- **Mock external dependencies**: APIs, routers, third-party libraries
- **Test user workflows**: E2E tests for critical paths
- **Use `data-testid` sparingly**: Prefer semantic queries

### Code Review

Tests must be reviewed for:
- [ ] Correct behavior verification
- [ ] Appropriate test type (unit vs integration)
- [ ] Clear test names
- [ ] No flaky tests (timing issues)
- [ ] Proper mocking
- [ ] Edge cases covered

## Continuous Integration

### CI Requirements

All tests must pass before merge:
- Backend: `./gradlew test`
- Frontend: `npm run test`
- Linting: Zero errors/warnings
- Build: Must succeed

### Test Failures

When tests fail in CI:
1. Reproduce locally
2. Fix the issue (not the test)
3. Add regression test if needed
4. Verify all tests pass
5. Push fix

## Resources

- [JUnit 5 Documentation](https://junit.org/junit5/docs/current/user-guide/)
- [MockK Documentation](https://mockk.io/)
- [Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Vitest Documentation](https://vitest.dev/)
- [Playwright Documentation](https://playwright.dev/)
