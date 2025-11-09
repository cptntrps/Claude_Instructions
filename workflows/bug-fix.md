# Bug Fix Workflow

Framework for investigating and resolving bugs.

---

## 🎯 Core Principle

**Understand before fixing.** Bugs are symptoms—find the root cause, don't just patch symptoms.

---

## 🚨 Non-Negotiables

Before marking a bug as fixed:

- ✋ **Create a test that reproduces the bug** (if feasible)
- ✋ **Verify the test fails before your fix** (proves you found the issue)
- ✋ **Verify the test passes after your fix** (proves fix works)
- ✋ **Run full test suite** (ensure no regressions)
- ✋ **No new linting errors**

---

## 📋 Framework

### 1. Understand the Bug

**Gather information:**
- Read the full bug report/issue
- Understand expected vs. actual behavior
- Note reproduction steps
- Check error messages, stack traces, logs
- Identify affected environment (dev, staging, production)

**Questions to answer:**
- What should happen?
- What actually happens?
- How do I reproduce it?
- Is this a regression (worked before)?
- How many users are affected?

**Tools:**
```
- Read issue/ticket
- Check relevant logs if available
- Review related test files
```

---

### 2. Locate the Issue

**Exploration strategies:**

**Strategy A - Error-Driven (if you have a stack trace):**
```
1. Find the file/line from stack trace
2. Read that file
3. Trace backwards to find root cause
```

**Strategy B - Feature-Driven (no stack trace):**
```
1. Identify which feature/module is affected
2. Use Task/Explore to find relevant files
3. Read key files (controllers, services, etc.)
4. Search for related functions with Grep
```

**Strategy C - Test-Driven:**
```
1. Find existing tests for the feature
2. Read test files to understand expected behavior
3. Read implementation files
4. Identify discrepancy
```

**Tools:**
```
- Grep for error messages or function names
- Task/Explore for unfamiliar code areas
- Read files in parallel once identified
```

---

### 3. Reproduce the Bug

**Create a failing test (when possible):**

```javascript
// Example - bug: user can login with unverified email

describe('Authentication', () => {
  it('should reject login for unverified email', async () => {
    const user = await createUser({ emailVerified: false });

    const response = await request(app)
      .post('/api/auth/login')
      .send({ email: user.email, password: 'password' });

    expect(response.status).toBe(403);
    expect(response.body.error).toMatch(/email not verified/i);
  });
});
```

**Verify test fails:**
```bash
npm test -- login.test.ts
```

**If can't write test:**
- Document reproduction steps
- Test manually if needed
- Note why automated test isn't feasible

---

### 4. Identify Root Cause

**Debug strategies:**

**Add logging (temporarily):**
```javascript
console.log('User object:', JSON.stringify(user, null, 2));
console.log('Email verified:', user.emailVerified);
```

**Trace execution flow:**
- Follow the code path from entry point
- Identify where behavior diverges from expected
- Check conditionals, loops, async operations

**Common bug patterns:**
- Null/undefined values
- Off-by-one errors
- Race conditions (async)
- Type coercion issues
- Missing validation
- Incorrect conditional logic
- Scope issues

**Questions to answer:**
- Why does the current code produce the wrong result?
- What assumption was incorrect?
- Is this a logic error, data error, or timing error?

---

### 5. Implement the Fix

**Principles:**
- Fix the root cause, not the symptom
- Minimal scope - change only what's necessary
- Follow existing patterns in the codebase
- Consider edge cases

**Example:**
```javascript
// Before (bug)
async function login(email, password) {
  const user = await User.findByEmail(email);
  if (user && await user.verifyPassword(password)) {
    return generateToken(user);
  }
  throw new Error('Invalid credentials');
}

// After (fixed)
async function login(email, password) {
  const user = await User.findByEmail(email);

  if (!user || !await user.verifyPassword(password)) {
    throw new Error('Invalid credentials');
  }

  // FIX: Check email verification
  if (!user.emailVerified) {
    throw new Error('Email not verified');
  }

  return generateToken(user);
}
```

**Remove debug logging** added during investigation.

---

### 6. Verify the Fix

**Run the reproduction test:**
```bash
npm test -- login.test.ts
```

**Verify it now passes.**

**Run full test suite:**
```bash
npm test
```

**Verify no regressions.**

**Check for similar bugs:**
- Search codebase for similar patterns
- If found, fix them too (or create tickets)

**Example:**
```
Fixed login.ts - now search for other auth methods
that might have the same issue:
- signup.ts
- passwordReset.ts
- oauth.ts
```

---

### 7. Document and Commit

**Commit message format:**
```
fix(scope): brief description

Longer explanation of the bug and fix (optional).
Fixes #issue-number
```

**Example:**
```bash
git add src/auth/login.ts src/auth/login.test.ts

git commit -m "$(cat <<'EOF'
fix(auth): require email verification before login

Previously, users with unverified emails could log in.
Now returns 403 error if email is not verified.

Fixes #456
EOF
)"
```

**Update issue/ticket:**
- Link to commit
- Explain what was wrong and how it's fixed
- Note if any follow-up work is needed

---

## 🎚️ Adaptive Approach

### Simple Bugs (typos, obvious errors)

**Streamlined process:**
```
1. Identify issue (obvious from error)
2. Fix it
3. Run tests
4. Commit

Skip: Reproduction test (obvious fix)
```

**Example:** Variable misspelled in one place.

---

### Complex Bugs (race conditions, data corruption)

**Thorough process:**
```
1. Deep investigation (logging, debugging)
2. Create comprehensive reproduction test
3. Possibly add multiple test cases
4. Implement fix
5. Extensive testing (unit, integration, manual)
6. Check for similar patterns
7. Document thoroughly
```

**Example:** Intermittent failure under load.

---

### Production Bugs (users affected NOW)

**Prioritize speed:**
```
1. Quick diagnosis
2. Implement minimal fix
3. Deploy hotfix
4. Create proper fix later if needed
5. Post-mortem analysis
```

**See:** [Rollback & Recovery](../advanced/rollback-recovery.md)

---

## ⚠️ Common Pitfalls

### Patching Symptoms

**❌ Bad:**
```javascript
// Crashes on undefined
// "Fix": Add defensive check
if (user && user.email) {
  sendEmail(user.email);
}
```

**✅ Good:**
```javascript
// Find why user is undefined in the first place
// Fix the root cause
```

---

### Incomplete Testing

**❌ Bad:**
```
Fixed the code, manually tested once, looks good.
```

**✅ Good:**
```
1. Added automated test
2. Verified test failed before fix
3. Verified test passes after fix
4. Ran full suite - no regressions
```

---

### Scope Creep

**❌ Bad:**
```
While fixing login bug, also refactored entire auth module,
renamed variables, reorganized files, updated styling...
```

**✅ Good:**
```
Fixed login bug only.
Created separate tickets for improvements noticed.
```

---

## 📊 Checklist

Before considering a bug fixed:

- [ ] Fully understand what's wrong and why
- [ ] Located root cause (not just symptom)
- [ ] Created reproduction test (if feasible)
- [ ] Implemented minimal fix
- [ ] Reproduction test passes
- [ ] Full test suite passes
- [ ] No linting errors
- [ ] Checked for similar bugs
- [ ] Committed with proper message
- [ ] Updated issue/ticket
- [ ] Considered if documentation needs update

---

## 💡 Examples

### Example 1: Frontend Bug

**Report:** "Submit button doesn't work on profile page"

**Process:**
```
1. Understand: Button click does nothing
2. Locate: Read ProfilePage.tsx, check onClick handler
3. Reproduce: Create test clicking submit button
4. Root cause: Form validation always returns false
5. Fix: Correct validation logic in validateProfile()
6. Verify: Test passes, full suite passes
7. Commit: fix(profile): correct form validation logic
```

---

### Example 2: API Bug

**Report:** "GET /api/users returns 500 error"

**Process:**
```
1. Understand: Read error logs, see SQL syntax error
2. Locate: Find query in users.controller.ts
3. Reproduce: Create test calling GET /api/users
4. Root cause: Missing WHERE clause in query
5. Fix: Add WHERE clause with proper escaping
6. Verify: Test passes, check for SQL injection vulnerability
7. Commit: fix(api): add WHERE clause to users query
```

---

### Example 3: Data Bug

**Report:** "User balances showing incorrect values"

**Process:**
```
1. Understand: Balance should be sum of transactions
2. Locate: calculateBalance() in account.service.ts
3. Reproduce: Create test with known transactions
4. Root cause: Not filtering by account ID
5. Fix: Add account ID filter to transaction query
6. Verify: Test with multiple accounts
7. Commit: fix(accounts): filter transactions by account ID
8. Check: Search for other queries missing account filter
```

---

**Remember:** A bug isn't fixed until it's tested, verified, and won't happen again. Take the time to understand and fix it properly.
