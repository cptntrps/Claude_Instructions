# Testing Standards

Requirements and best practices for test coverage and quality.

---

## 🚨 Non-Negotiables

- ✋ **All tests must pass** before committing
- ✋ **Minimum 70% code coverage** (80% for critical paths)
- ✋ **New features must have tests**
- ✋ **Bug fixes must have regression tests**
- ✋ **No skipped tests without documented reason**

---

## 📊 Coverage Requirements

### Overall Coverage: ≥ 70%

### By Category

- **Critical paths:** ≥ 80%
  - Authentication/authorization
  - Payment processing
  - Data integrity operations
  - Security features

- **Business logic:** ≥ 80%
  - Calculations
  - Validation
  - State management

- **API endpoints:** ≥ 70%
  - Happy paths
  - Error cases
  - Validation

- **UI components:** ≥ 60%
  - User interactions
  - State changes
  - Error states

- **Utilities:** ≥ 90%
  - Pure functions
  - Helpers

---

## ✅ What to Test

### Always Test
- Business logic and calculations
- Validation functions
- API endpoints (happy + error paths)
- Authentication/authorization
- Data transformations
- Edge cases and boundary conditions
- Error handling

### Sometimes Test
- UI components (focus on logic, not rendering)
- Integration between modules
- Database queries (integration tests)

### Don't Test
- Third-party libraries
- Framework internals
- Simple getters/setters
- Generated code

---

## 📝 Test Naming

### Format
```
describe('ComponentOrFunction', () => {
  it('should do something when condition', () => {
    // test
  });
});
```

### Examples

**✅ Good:**
```typescript
describe('validateEmail', () => {
  it('should return true for valid email addresses');
  it('should return false for emails without @ symbol');
  it('should return false for empty strings');
  it('should throw error for null or undefined');
});

describe('POST /api/users', () => {
  it('should create user and return 201 for valid data');
  it('should return 400 for invalid email format');
  it('should return 400 for password shorter than 8 characters');
  it('should return 401 if not authenticated');
});
```

**❌ Bad:**
```typescript
it('test email'); // Vague
it('works'); // Not descriptive
it('email validation'); // Doesn't specify expected behavior
```

---

## 🎯 Test Structure (AAA Pattern)

```typescript
it('should calculate discount for premium users', () => {
  // Arrange: Set up test data
  const user = { isPremium: true };
  const price = 100;

  // Act: Execute the function
  const result = calculateDiscount(price, user);

  // Assert: Verify the result
  expect(result).toBe(90);
});
```

---

## 📋 Test Checklist

### For Each Feature
- [ ] Happy path tested
- [ ] Error cases tested
- [ ] Edge cases tested (null, undefined, empty, boundary values)
- [ ] Input validation tested
- [ ] Authentication/authorization tested (if applicable)
- [ ] Integration points tested

### Test Quality
- [ ] Tests are independent (no shared state)
- [ ] Tests are deterministic (same result every time)
- [ ] Tests are fast (unit tests < 100ms)
- [ ] Tests have clear, descriptive names
- [ ] Tests follow AAA pattern
- [ ] No commented-out tests
- [ ] No console.logs in tests

---

## 🧪 Test Types

### Unit Tests (Most)
```typescript
// Test pure function
describe('calculateTax', () => {
  it('applies 7.25% tax for California', () => {
    expect(calculateTax(100, 'CA')).toBe(107.25);
  });

  it('applies 6% tax for other states', () => {
    expect(calculateTax(100, 'NY')).toBe(106);
  });
});
```

### Integration Tests (Some)
```typescript
// Test API endpoint with database
describe('POST /api/users', () => {
  it('creates user in database and returns 201', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ email: 'test@example.com', password: 'password123' });

    expect(response.status).toBe(201);

    const user = await db.users.findByEmail('test@example.com');
    expect(user).toBeDefined();
  });
});
```

### E2E Tests (Few)
```typescript
// Test complete user flow
test('user can sign up and create post', async ({ page }) => {
  await page.goto('/signup');
  await page.fill('[name="email"]', 'user@example.com');
  await page.click('button[type="submit"]');
  // ... complete workflow
});
```

---

## ⚠️ Common Issues

### Flaky Tests
**Problem:** Test passes sometimes, fails sometimes

**Causes:**
- Race conditions (async timing)
- Shared state between tests
- Depending on time/randomness
- External service dependency

**Solutions:**
```typescript
❌ Flaky: setTimeout guessing
it('updates after delay', async () => {
  updateAsync();
  await new Promise(r => setTimeout(r, 100)); // Brittle!
  expect(value).toBe('updated');
});

✅ Fixed: Properly await
it('updates after delay', async () => {
  await updateAsync();
  expect(value).toBe('updated');
});
```

### Slow Tests
**Problem:** Test suite takes too long

**Solutions:**
- Mock external services
- Use in-memory database for tests
- Parallelize test execution
- Focus on unit tests over E2E

### False Positives
**Problem:** Tests pass but code is broken

**Causes:**
- Testing implementation, not behavior
- Weak assertions
- Not covering actual use case

**Solution:**
```typescript
❌ Weak assertion
it('creates user', async () => {
  const result = await createUser(data);
  expect(result).toBeDefined(); // Could be anything!
});

✅ Strong assertion
it('creates user with correct data', async () => {
  const result = await createUser({
    email: 'test@example.com',
    name: 'Test User'
  });

  expect(result).toMatchObject({
    email: 'test@example.com',
    name: 'Test User',
    id: expect.any(String)
  });
});
```

---

## 🔧 Running Tests

```bash
# All tests
npm test

# Specific file
npm test -- userService.test.ts

# Watch mode
npm test -- --watch

# Coverage report
npm run test:coverage

# Only changed files
npm test -- --onlyChanged
```

---

**Remember:** Tests are your safety net. Invest in comprehensive testing to move fast with confidence.
