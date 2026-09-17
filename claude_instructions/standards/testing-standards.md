# Testing Standards

Requirements and best practices for test coverage and quality.

**Bug-fix scope:** When supporting [Bug Fix Workflow v2](../workflows/bug-fix.md),
follow its verification and failure-classification contract instead of conflicting
defaults below. An adequate existing regression test is sufficient. Assertion
choice follows the behavior being tested; existence checks are not inherently
invalid. Coverage thresholds and mutation testing apply when the project requires
them or the risk justifies them. No independent failure waives a required gate.

---

## 🚨 Non-Negotiables

- ✋ **All tests must pass** before committing
- ✋ **Minimum 70% code coverage** (80% for critical paths)
- ✋ **New features must have tests**
- ✋ **Bug fixes must have regression tests**
- ✋ **No skipped tests without documented reason**

---

## 🔴 FAIL-FIRST TESTING PROTOCOL (Anti-Theater)

**NEW in v1.3.2:** This protocol is NON-NEGOTIABLE for preventing testing theater.

### The Problem: Testing Theater

**Testing theater** occurs when tests pass without actually validating behavior:

```typescript
// TESTING THEATER - This test always passes
it('validates email format', () => {
  const email = 'test@example.com';
  validateEmail(email); // Executes code
  expect(true).toBe(true); // Always passes!
});

// This achieves 100% code coverage but validates NOTHING
```

**Evidence from framework history:** v1.4.0 claimed "100% test pass rate" but only tested syntax, not functionality.

### The Solution: Fail-First Protocol

**MANDATORY for all new tests:**

```
1. Write the test FIRST (before implementation)
2. Run the test → It MUST FAIL
3. Implement the code
4. Run the test → It MUST PASS
```

### Why Tests Must Fail First

**If a test doesn't fail before implementation, it's not testing anything.**

**Example:**

```typescript
// Step 1: Write test FIRST
describe('calculateDiscount', () => {
  it('applies 10% discount for premium users', () => {
    const result = calculateDiscount(100, { isPremium: true });
    expect(result).toBe(90);
  });
});

// Step 2: Run test → MUST FAIL
// Error: calculateDiscount is not defined
// ✓ Good! Test will catch if function is missing

// Step 3: Implement (incorrectly first)
function calculateDiscount(price, user) {
  return price; // BUG: Not checking isPremium
}

// Run test → MUST FAIL
// Expected: 90, Received: 100
// ✓ Good! Test caught the bug

// Step 4: Fix implementation
function calculateDiscount(price, user) {
  if (user.isPremium) {
    return price * 0.9;
  }
  return price;
}

// Run test → MUST PASS
// ✓ Good! Test validates behavior
```

### Fail-First Checklist

Before committing ANY test:

- [ ] Test was written before implementation
- [ ] Test failed when code was missing
- [ ] Test failed when code was wrong
- [ ] Test passes when code is correct
- [ ] Can explain what bug this test catches

**If you cannot make a test fail first, DELETE IT and write a real test.**

### Exception: Existing Code

For legacy code without tests:

```
1. Write test for existing behavior
2. Run test → Should PASS (code already works)
3. Introduce deliberate bug
4. Run test → MUST FAIL (proves test works)
5. Fix bug (restore original code)
6. Run test → PASS again

This validates the test catches real bugs.
```

### Red-Green-Refactor Cycle

**TDD (Test-Driven Development) workflow:**

```
🔴 RED: Write failing test
   ↓
🟢 GREEN: Make test pass (simplest code)
   ↓
🔵 REFACTOR: Improve code (tests still pass)
   ↓
🔴 RED: Next test...
```

**Benefits:**
- Tests validate actual behavior (not theater)
- Code is testable by design
- Complete test coverage (no gaps)
- Tests document requirements

### Common Violations

**❌ Test Written After Implementation**
```typescript
// Already implemented calculateTax()
// Now writing test...
it('calculates tax correctly', () => {
  expect(calculateTax(100, 'CA')).toBe(107.25); // Passes immediately
});
```
**Problem:** Test never failed → Don't know if it works

**❌ Test That Can't Fail**
```typescript
it('creates user', async () => {
  await createUser(data);
  expect(true).toBe(true); // Tautology
});
```
**Problem:** This test will pass even if createUser throws an error

**✅ Proper Fail-First**
```typescript
// Write test FIRST
it('creates user in database', async () => {
  const user = await createUser({ email: 'test@example.com' });
  expect(user.id).toBeDefined();
  expect(user.email).toBe('test@example.com');
});

// Run → FAILS (function doesn't exist yet)
// Implement → Test PASSES
// Validates behavior correctly
```

### Framework Integration

**Context Anchor in testing.md enforces this:**

```
[CONTEXT ANCHOR: After Writing Test]
- CRITICAL: Run test and verify it FAILS
- If test passes before implementation: Testing theater
- If test doesn't fail: Revise test or implementation
```

**Framework Heartbeat checks:**
- Every 20 messages, verify no weak assertions
- Before commit, validate fail-first protocol followed
- Drift check reports testing theater violations

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

## ⚡ ASSERTION STRENGTH VALIDATION (Anti-Theater)

**NEW in v1.3.2:** Weak assertions are the #1 cause of testing theater.

### The Problem: Weak Assertions

**Weak assertions** almost always pass, regardless of code correctness:

```typescript
// These assertions test NOTHING
expect(result).toBeDefined();     // Passes unless result is undefined
expect(result).toBeTruthy();      // Passes for any truthy value
expect(result).not.toBeNull();    // Passes unless null
expect(true).toBe(true);          // Always passes (tautology)
```

### Forbidden Weak Assertions

**NEVER use these in tests:**

| Assertion | Why It's Weak | What Happens |
|-----------|---------------|--------------|
| `toBeDefined()` | Only fails if `undefined` | Passes for `null`, `false`, `0`, `''`, `{}`, `[]` |
| `toBeTruthy()` | Only fails for falsy values | Passes for `1`, `"error"`, `{}`, `[]` |
| `toBeFalsy()` | Only fails for truthy values | Passes for `0`, `''`, `null`, `undefined` |
| `not.toBeNull()` | Only fails if `null` | Passes for everything else |
| `not.toBeUndefined()` | Only fails if `undefined` | Passes for everything else |
| `expect(true).toBe(true)` | Tautology | Always passes |
| `expect(false).toBe(false)` | Tautology | Always passes |

### Required: Strong Assertions

**ALWAYS use specific assertions:**

| Strong Assertion | What It Validates | Use When |
|------------------|-------------------|----------|
| `toBe(value)` | Exact primitive value | Numbers, strings, booleans |
| `toEqual(object)` | Deep object equality | Objects, arrays |
| `toMatchObject(partial)` | Partial object match | Subset of properties |
| `toThrow(error)` | Specific error thrown | Error handling |
| `toHaveLength(n)` | Exact array/string length | Collections |
| `toContain(item)` | Array contains item | List validation |
| `toMatch(regex)` | String pattern match | Format validation |
| `toBeGreaterThan(n)` | Numeric comparison | Calculations |
| `toHaveBeenCalledWith(...)` | Mock called with args | Function calls |

### Examples: Weak vs. Strong

**❌ Weak (Testing Theater):**
```typescript
it('creates user', async () => {
  const user = await createUser(data);
  expect(user).toBeDefined(); // Could be anything!
});

it('validates email', () => {
  const result = validateEmail('test@example.com');
  expect(result).toBeTruthy(); // Could be 1, "yes", {}, etc.
});

it('calculates total', () => {
  const total = calculateTotal(items);
  expect(total).not.toBeNull(); // Could be 0, -1, "error", etc.
});
```

**✅ Strong (Real Testing):**
```typescript
it('creates user with correct properties', async () => {
  const user = await createUser({
    email: 'test@example.com',
    name: 'Test User'
  });

  expect(user).toMatchObject({
    email: 'test@example.com',
    name: 'Test User',
    id: expect.any(String),
    createdAt: expect.any(Date)
  });
});

it('validates email format correctly', () => {
  expect(validateEmail('test@example.com')).toBe(true);
  expect(validateEmail('invalid')).toBe(false);
  expect(validateEmail('')).toBe(false);
  expect(validateEmail(null)).toBe(false);
});

it('calculates total price correctly', () => {
  const items = [
    { price: 10, quantity: 2 },
    { price: 5, quantity: 3 }
  ];

  expect(calculateTotal(items)).toBe(35); // 10*2 + 5*3
});
```

### Pre-Commit Validation

**Before committing tests, run these checks:**

```bash
# Search for weak assertions in test files
grep -r "toBeDefined()" tests/       # Should return NOTHING
grep -r "toBeTruthy()" tests/        # Should return NOTHING
grep -r "toBeFalsy()" tests/         # Should return NOTHING
grep -r "not.toBeNull()" tests/      # Should return NOTHING
grep -r "expect(true)" tests/        # Should return NOTHING
```

**If found:** Replace with strong assertions before committing.

### Assertion Strength Checklist

For each test assertion:

- [ ] Does this assertion test a specific value/behavior?
- [ ] Would this assertion fail if the code is broken?
- [ ] Can I explain what bug this assertion catches?
- [ ] Is this the most specific assertion possible?

**If you answer "no" to any question:** Strengthen the assertion.

### Multiple Assertions Per Test

**Good practice:** Multiple strong assertions in one test

```typescript
it('processes order correctly', async () => {
  const order = await processOrder(orderData);

  // Multiple strong assertions
  expect(order.status).toBe('confirmed');
  expect(order.total).toBe(100);
  expect(order.items).toHaveLength(3);
  expect(order.customer.email).toBe('test@example.com');
  expect(order.createdAt).toBeInstanceOf(Date);
});
```

**Each assertion validates a specific aspect of behavior.**

### Common Weak Assertion Patterns

**Pattern 1: Just checking existence**
```typescript
❌ expect(result).toBeDefined();
✅ expect(result).toBe(expectedValue);
```

**Pattern 2: Just checking truthiness**
```typescript
❌ expect(isValid).toBeTruthy();
✅ expect(isValid).toBe(true);
```

**Pattern 3: Just checking not-null**
```typescript
❌ expect(user).not.toBeNull();
✅ expect(user).toMatchObject({ id: expect.any(String), name: 'John' });
```

**Pattern 4: Tautology**
```typescript
❌ expect(true).toBe(true);
✅ Remove this test - it tests nothing
```

### Framework Integration

**Context Anchor enforcement:**

```
[CONTEXT ANCHOR: Testing Complete]
- Verify no weak assertions (toBeDefined, toBeTruthy)
- Check every assertion validates specific behavior
- Ensure assertions would fail if code broken
```

**Framework Heartbeat checks:**
- Scan test files for weak assertion patterns
- Report weak assertions as testing theater
- Block commit if weak assertions detected

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

## 🧬 MUTATION TESTING (Critical Paths Only)

**NEW in v1.3.2:** Mutation testing validates that tests actually catch bugs.

### What is Mutation Testing?

**Mutation testing** introduces deliberate bugs into code to verify tests detect them.

**Concept:**
```
1. Tests pass with correct code ✓
2. Introduce deliberate bug (mutation)
3. Tests MUST fail
4. If tests still pass → Tests are inadequate
```

### When to Use Mutation Testing

**REQUIRED for critical paths:**
- Authentication/authorization
- Payment processing
- Data integrity operations
- Security features
- Financial calculations

**OPTIONAL for other code:**
- Nice to have, but not mandatory
- Focus mutation testing where bugs are most costly

### How to Perform Mutation Testing

**Process:**

```typescript
// Step 1: Original code (correct)
function authenticateUser(username, password) {
  const user = findUser(username);
  if (!user) return false;
  return bcrypt.compare(password, user.hashedPassword);
}

// Tests pass ✓

// Step 2: Mutation 1 - Remove password check
function authenticateUser(username, password) {
  const user = findUser(username);
  if (!user) return false;
  return true; // BUG: Always returns true!
}

// Run tests → MUST FAIL
// If tests pass: Authentication tests are inadequate!

// Step 3: Mutation 2 - Remove user check
function authenticateUser(username, password) {
  const user = findUser(username);
  // Removed: if (!user) return false;
  return bcrypt.compare(password, user.hashedPassword); // Crashes if user is null
}

// Run tests → MUST FAIL
// If tests pass: Edge case testing is inadequate!

// Step 4: Fix mutations, verify tests pass again
```

### Common Mutations to Test

**For each critical function, test these mutations:**

**1. Remove conditions**
```typescript
Original: if (x > 0) return x;
Mutation: return x; // Removed condition
Expected: Tests should fail
```

**2. Change operators**
```typescript
Original: if (x > 0)
Mutation: if (x >= 0) // Changed > to >=
Expected: Boundary tests should fail
```

**3. Change constants**
```typescript
Original: return price * 0.9; // 10% discount
Mutation: return price * 1.0; // No discount
Expected: Calculation tests should fail
```

**4. Remove return statements**
```typescript
Original: if (error) return null;
Mutation: if (error) {} // Removed return
Expected: Error handling tests should fail
```

**5. Negate conditions**
```typescript
Original: if (isValid)
Mutation: if (!isValid)
Expected: Validation tests should fail
```

### Mutation Testing Example

**Code:** Password strength validator

```typescript
function isPasswordStrong(password) {
  if (password.length < 8) return false;
  if (!/[A-Z]/.test(password)) return false;
  if (!/[a-z]/.test(password)) return false;
  if (!/[0-9]/.test(password)) return false;
  return true;
}
```

**Tests:**
```typescript
describe('isPasswordStrong', () => {
  it('accepts strong password', () => {
    expect(isPasswordStrong('Test1234')).toBe(true);
  });

  it('rejects short password', () => {
    expect(isPasswordStrong('Test12')).toBe(false);
  });

  it('rejects password without uppercase', () => {
    expect(isPasswordStrong('test1234')).toBe(false);
  });

  it('rejects password without lowercase', () => {
    expect(isPasswordStrong('TEST1234')).toBe(false);
  });

  it('rejects password without numbers', () => {
    expect(isPasswordStrong('TestTest')).toBe(false);
  });
});
```

**Mutations to try:**
```typescript
// Mutation 1: Change length requirement
if (password.length < 6) return false; // Changed 8 to 6

// Run tests → Should FAIL (short password test)
// ✓ Tests caught this bug

// Mutation 2: Remove uppercase check
// if (!/[A-Z]/.test(password)) return false; // Commented out

// Run tests → Should FAIL (no uppercase test)
// ✓ Tests caught this bug

// Mutation 3: Change lowercase regex
if (!/[a-zA-Z]/.test(password)) return false; // Changed to any letter

// Run tests → Should FAIL (no lowercase test)
// ✓ Tests caught this bug
```

**If any mutation passes → Add more tests**

### Mutation Testing Tools

**Automated mutation testing:**

```bash
# JavaScript/TypeScript
npm install --save-dev stryker-cli
npx stryker run

# Python
pip install mutpy
mutpy --target mymodule --unit-test tests

# Java
# Maven: Add Pitest plugin
mvn org.pitest:pitest-maven:mutationCoverage
```

**Manual mutation testing:**
- Introduce bugs manually
- Run test suite
- Verify tests fail
- Revert bugs

### Mutation Score

**Mutation score = (Killed mutations / Total mutations) × 100**

**Example:**
- Introduced 10 mutations
- 8 caught by tests (tests failed)
- 2 not caught (tests still passed)
- Mutation score: 80%

**Target for critical paths:** ≥90% mutation score

### Documentation

**Document in test files:**

```typescript
// tests/auth.test.ts

/**
 * MUTATION TESTING RESULTS
 *
 * Date: 2025-11-10
 * Mutations tested: 5
 * Mutations caught: 5
 * Mutation score: 100%
 *
 * Tested mutations:
 * 1. Removed password check → Test failed ✓
 * 2. Removed user existence check → Test failed ✓
 * 3. Always return true → Test failed ✓
 * 4. Skip bcrypt comparison → Test failed ✓
 * 5. Return wrong user → Test failed ✓
 */
```

### When NOT to Use Mutation Testing

**Skip mutation testing for:**
- Simple getters/setters
- Trivial formatting functions
- Non-critical utility code
- Third-party library wrappers

**Focus mutation testing on high-risk code where bugs are expensive.**

---

## 🎭 TESTING THEATER DETECTION GUIDE

**NEW in v1.3.2:** Identify and eliminate testing theater in your test suite.

### What is Testing Theater?

**Testing theater** is when tests give the appearance of quality assurance without actually validating behavior.

**Characteristics:**
- Tests always pass (never see failures during development)
- High coverage but bugs still reach production
- Tests don't fail when code is broken
- False sense of security

### Warning Signs of Testing Theater

**🚨 Red Flag 1: All Tests Always Pass**

```
Problem: Never see red during development
Reality: Real TDD shows failures frequently
Diagnosis: Tests aren't validating anything
```

**If you never see tests fail, you're doing testing theater.**

**🚨 Red Flag 2: High Coverage, Low Confidence**

```
Problem: 90% coverage but still finding bugs in production
Reality: Coverage measures execution, not validation
Diagnosis: Weak assertions or missing edge cases
```

**Coverage ≠ Quality**

**🚨 Red Flag 3: Weak Assertions Everywhere**

```typescript
// Count these patterns in your test files:
grep -c "toBeDefined()" tests/**/*.test.ts
grep -c "toBeTruthy()" tests/**/*.test.ts
grep -c "expect(true)" tests/**/*.test.ts

// If count > 10% of assertions: Testing theater alert
```

**🚨 Red Flag 4: Tests Don't Catch Bugs**

```
Scenario: Bug found in code
Action: Check if tests exist for that code
Result: Tests exist AND pass
Diagnosis: Tests are theater (not testing the right thing)
```

**If tests don't catch bugs, what's the point?**

**🚨 Red Flag 5: Can't Explain What Test Validates**

```
Question: "What bug does this test catch?"
Answer: "Umm... it makes sure the code runs?"
Diagnosis: Test validates execution, not behavior
```

**Every test should answer: "What specific bug does this catch?"**

### Testing Theater Diagnostic Checklist

**Run this checklist on your test suite:**

```
□ Have tests failed during development? (Should be YES)
□ Do tests use specific assertions? (toBe, toEqual, not toBeDefined)
□ Can you explain what each test validates?
□ Do tests catch bugs you introduce?
□ Are edge cases tested? (null, undefined, empty, boundaries)
□ Do tests validate behavior, not just execution?
□ Would tests fail if implementation is wrong?
```

**If you answered NO to any:** You have testing theater.

### Converting Theater to Real Tests

**Theater Test:**
```typescript
it('processes order', async () => {
  const order = await processOrder(orderData);
  expect(order).toBeDefined();
});
```

**Problems:**
- ❌ Only checks if result exists
- ❌ Doesn't validate what order contains
- ❌ Would pass even if order is wrong
- ❌ Can't explain what bug this catches

**Real Test:**
```typescript
it('processes order with correct status and total', async () => {
  const orderData = {
    items: [{ id: '1', price: 10, quantity: 2 }],
    customerId: 'cust-123'
  };

  const order = await processOrder(orderData);

  expect(order).toMatchObject({
    status: 'pending',
    total: 20,
    customerId: 'cust-123',
    items: expect.arrayContaining([
      expect.objectContaining({ id: '1', quantity: 2 })
    ])
  });
});
```

**Improvements:**
- ✓ Validates specific values
- ✓ Checks status is correct
- ✓ Verifies total calculation
- ✓ Confirms customer association
- ✓ Ensures items are preserved
- ✓ Would fail if any of these are wrong

**Can explain:** "This test catches bugs where orders are created with wrong status, incorrect totals, missing customer IDs, or lost items."

### Theater Detection Commands

**Scan for weak assertions:**

```bash
#!/bin/bash
# Run this script to detect testing theater

echo "🎭 Testing Theater Detection Report"
echo "===================================="
echo ""

echo "Weak Assertions Found:"
echo "  toBeDefined:     $(grep -r "toBeDefined()" tests/ | wc -l)"
echo "  toBeTruthy:      $(grep -r "toBeTruthy()" tests/ | wc -l)"
echo "  toBeFalsy:       $(grep -r "toBeFalsy()" tests/ | wc -l)"
echo "  not.toBeNull:    $(grep -r "not.toBeNull()" tests/ | wc -l)"
echo "  Tautologies:     $(grep -r "expect(true)" tests/ | wc -l)"
echo ""

TOTAL_WEAK=$(( $(grep -r "toBeDefined()" tests/ | wc -l) + \
               $(grep -r "toBeTruthy()" tests/ | wc -l) + \
               $(grep -r "toBeFalsy()" tests/ | wc -l) + \
               $(grep -r "not.toBeNull()" tests/ | wc -l) + \
               $(grep -r "expect(true)" tests/ | wc -l) ))

TOTAL_TESTS=$(grep -r "it('\\|it(\"" tests/ | wc -l)

THEATER_PERCENTAGE=$(( $TOTAL_WEAK * 100 / $TOTAL_TESTS ))

echo "Theater Risk: $THEATER_PERCENTAGE%"

if [ $THEATER_PERCENTAGE -gt 20 ]; then
  echo "🚨 HIGH RISK: Significant testing theater detected"
elif [ $THEATER_PERCENTAGE -gt 10 ]; then
  echo "⚠️  MODERATE RISK: Some testing theater present"
else
  echo "✅ LOW RISK: Minimal testing theater"
fi
```

### Theater Elimination Strategy

**Phase 1: Identify (Week 1)**
1. Run theater detection scan
2. Identify weak assertion patterns
3. List tests that can't explain what they validate

**Phase 2: Strengthen (Week 2-3)**
1. Replace weak assertions with strong ones
2. Add fail-first validation for new tests
3. Apply mutation testing to critical paths

**Phase 3: Maintain (Ongoing)**
1. Pre-commit hooks block weak assertions
2. Code review checks for testing theater
3. Regular mutation testing for critical code

### Prevention: Framework Integration

**This framework prevents testing theater through:**

**1. Fail-First Protocol** (Non-negotiable)
- Forces tests to fail before implementation
- Proves tests validate behavior

**2. Assertion Strength Validation** (Pre-commit check)
- Blocks weak assertions
- Requires specific value testing

**3. Context Anchors** (Automated reminders)
- Reminds to verify tests fail first
- Checks for weak assertions at key points

**4. Mutation Testing** (Critical paths)
- Validates tests catch real bugs
- Measures test adequacy objectively

**5. Framework Heartbeat** (Every 20 messages)
- Scans for weak assertion patterns
- Reports testing theater as framework drift
- Self-corrects before commit

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
