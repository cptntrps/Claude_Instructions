# Code Review Workflow

Framework for analyzing code quality and suggesting improvements.

---

## 🎯 Core Principle

**Be objective, helpful, and constructive.** Focus on facts, not opinions. Prioritize correctness, security, and maintainability.

---

## 🚨 Non-Negotiables

When reviewing code, ALWAYS flag:

- ✋ **Security vulnerabilities** (injection, XSS, exposed secrets, broken auth)
- ✋ **Bugs or incorrect logic**
- ✋ **Missing error handling** for external calls
- ✋ **Missing input validation** for user inputs
- ✋ **Test failures or missing tests** for critical code
- ✋ **Breaking changes** without clear justification
- ✋ **Performance issues** that will cause problems at scale (N+1 queries, memory leaks)

---

## 📋 Framework

### 1. Understand the Context

**Before reviewing, understand:**
- What is the purpose of this code?
- What problem does it solve?
- Is this a bug fix, feature, or refactor?
- What files are changed and why?

**Read:**
- PR description or commit messages
- Related issue/ticket
- Changed files
- Relevant tests

---

### 2. Review Categories

Review in this priority order:

#### 🔴 Critical (Must Fix)

**Correctness:**
- Logic errors or bugs
- Incorrect algorithms
- Race conditions
- Memory leaks
- Incorrect type usage

**Security:**
- SQL/NoSQL injection vulnerabilities
- XSS vulnerabilities
- Authentication/authorization bypassed
- Secrets hardcoded or exposed
- Sensitive data logged
- Missing CSRF protection
- Insecure dependencies

**Example:**
```typescript
❌ CRITICAL: SQL Injection vulnerability
const query = `SELECT * FROM users WHERE email = '${email}'`;

✅ Fix: Use parameterized query
const query = 'SELECT * FROM users WHERE email = ?';
db.query(query, [email]);
```

---

#### 🟡 Important (Should Fix)

**Error Handling:**
- Missing try-catch for external calls
- Poor error messages
- Errors not logged
- Failing silently

**Testing:**
- Missing tests for new functionality
- Tests don't cover edge cases
- Tests are flaky or unreliable

**Performance:**
- N+1 query problems
- Inefficient algorithms (O(n²) when O(n) possible)
- Unnecessary database calls
- Missing indexes

**Example:**
```typescript
⚠️ IMPORTANT: N+1 query problem
const users = await db.users.findAll();
for (const user of users) {
  user.posts = await db.posts.findByUserId(user.id); // N queries!
}

✅ Fix: Use eager loading
const users = await db.users.findAll({ include: ['posts'] });
```

---

#### 🔵 Suggestions (Nice to Have)

**Code Quality:**
- Unclear naming
- Overly complex functions
- Duplicated code
- Missing comments for complex logic
- Inconsistent style

**Best Practices:**
- Magic numbers (use constants)
- Long functions (break down)
- Too many parameters
- Tight coupling

**Example:**
```typescript
💡 SUGGESTION: Use descriptive constant
if (user.age < 13) { // Magic number

✅ Better:
const MINIMUM_AGE = 13;
if (user.age < MINIMUM_AGE) {
```

---

### 3. Review Process

**Step 1: Read all changes**
```
Read all modified files to understand scope
```

**Step 2: Check tests**
```
- Do tests exist?
- Do they pass?
- Do they cover new functionality?
- Do they cover edge cases?
```

**Step 3: Security review**
```
Check security checklist:
- Input validation
- Output escaping
- Authentication/authorization
- SQL/XSS injection prevention
- No exposed secrets
```

**Step 4: Logic review**
```
- Is the logic correct?
- Are edge cases handled?
- Is error handling appropriate?
- Are there race conditions?
```

**Step 5: Performance review**
```
- Any obvious performance issues?
- Database queries optimized?
- Caching where appropriate?
- Proper pagination?
```

**Step 6: Code quality**
```
- Clear naming?
- Appropriate complexity?
- Follows project conventions?
- Well documented?
```

---

### 4. Provide Feedback

**Structure feedback by severity:**

```markdown
## Critical Issues ❌

1. **Security: SQL Injection in login.ts:45**
   - Current: `SELECT * FROM users WHERE email = '${email}'`
   - Fix: Use parameterized query
   - Impact: Allows database compromise

2. **Bug: Null pointer in profile.ts:78**
   - `user.email.toLowerCase()` will crash if email is null
   - Fix: Add null check or optional chaining

## Important Issues ⚠️

3. **Performance: N+1 query in getUserPosts()**
   - Making separate query for each user
   - Fix: Add eager loading with include

4. **Missing Tests: No tests for new validation logic**
   - Add tests for edge cases (empty string, null, invalid format)

## Suggestions 💡

5. **Code Quality: Extract validation logic**
   - Current: Inline validation is duplicated
   - Suggestion: Extract to `validateEmail()` function

6. **Naming: Variable `x` is unclear**
   - Rename to `userCount` for clarity
```

**Be specific:**
- Point to exact file:line
- Explain what's wrong
- Explain why it's wrong
- Suggest how to fix it
- Include code examples

---

### 5. Example Review

**Code being reviewed:**
```typescript
// src/api/users.ts
export async function createUser(req: Request, res: Response) {
  const { email, password, age } = req.body;

  const user = await db.query(
    `INSERT INTO users (email, password, age) VALUES ('${email}', '${password}', ${age})`
  );

  res.json(user);
}
```

**Review feedback:**

```markdown
## Critical Issues ❌

1. **Security: SQL Injection (users.ts:5-7)**
   Problem: User input directly interpolated into SQL query
   Impact: Attacker can execute arbitrary SQL commands
   Fix:
   ```typescript
   const user = await db.query(
     'INSERT INTO users (email, password, age) VALUES (?, ?, ?)',
     [email, password, age]
   );
   ```

2. **Security: Password stored in plaintext (users.ts:6)**
   Problem: Password not hashed before storage
   Impact: Database breach exposes all passwords
   Fix:
   ```typescript
   const hashedPassword = await bcrypt.hash(password, 10);
   ```

3. **Missing: No input validation (users.ts:4)**
   Problem: No validation on email format, password strength, age range
   Impact: Invalid data can be stored
   Fix: Add validation schema (Zod, Joi, etc.)

4. **Missing: No authentication check**
   Problem: Endpoint is publicly accessible
   Impact: Anyone can create users
   Fix: Add authentication middleware

## Important Issues ⚠️

5. **Error Handling: No try-catch**
   Problem: Database errors will crash the server
   Fix: Wrap in try-catch and return appropriate error response

6. **Missing Tests**
   Problem: No tests for this endpoint
   Fix: Add tests for happy path and validation failures

## Suggestions 💡

7. **Response: Return 201 instead of 200**
   Suggestion: Use correct HTTP status for resource creation

8. **Security: Don't return full user object**
   Suggestion: Omit password from response

## Corrected Version

```typescript
export async function createUser(req: Request, res: Response) {
  try {
    // Validate input
    const schema = z.object({
      email: z.string().email(),
      password: z.string().min(8),
      age: z.number().int().min(13).max(120),
    });

    const data = schema.parse(req.body);

    // Hash password
    const hashedPassword = await bcrypt.hash(data.password, 10);

    // Create user with parameterized query
    const user = await db.query(
      'INSERT INTO users (email, password, age) VALUES (?, ?, ?) RETURNING id, email, age',
      [data.email, hashedPassword, data.age]
    );

    res.status(201).json(user);
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({ error: error.errors });
    }

    logger.error('Failed to create user', error);
    res.status(500).json({ error: 'Failed to create user' });
  }
}
```
```

---

## 📊 Review Checklist

### Security ✓
- [ ] No SQL/NoSQL injection vulnerabilities
- [ ] No XSS vulnerabilities
- [ ] Authentication/authorization enforced
- [ ] No hardcoded secrets
- [ ] Input validation present
- [ ] Output properly escaped
- [ ] Sensitive data not logged
- [ ] HTTPS enforced (if applicable)

### Correctness ✓
- [ ] Logic is correct
- [ ] Edge cases handled
- [ ] No obvious bugs
- [ ] Type safety enforced
- [ ] Error handling implemented

### Testing ✓
- [ ] Tests exist for new code
- [ ] Tests cover edge cases
- [ ] All tests pass
- [ ] Coverage is adequate

### Performance ✓
- [ ] No N+1 queries
- [ ] Efficient algorithms used
- [ ] Proper indexing (database)
- [ ] Caching where appropriate
- [ ] Pagination for large datasets

### Code Quality ✓
- [ ] Clear, descriptive names
- [ ] Functions are focused and small
- [ ] No code duplication
- [ ] Appropriate comments
- [ ] Follows project conventions
- [ ] No linting errors

### Documentation ✓
- [ ] Public APIs documented
- [ ] Complex logic explained
- [ ] README updated if needed
- [ ] Commit message is clear

---

## ⚠️ Review Anti-Patterns

### Don't Nitpick Style

**❌ Bad:**
```
I prefer double quotes instead of single quotes.
Can you rename this variable to match my style?
I would structure this differently.
```

**✅ Good:**
```
Code follows project's ESLint config. Style is consistent.
```

---

### Don't Be Vague

**❌ Bad:**
```
This code doesn't look good.
This might cause issues.
Consider refactoring this.
```

**✅ Good:**
```
This function has cyclomatic complexity of 15 (threshold is 10).
Suggest extracting validation logic to separate function.
This reduces complexity and improves testability.
```

---

### Don't Overwhelm

**❌ Bad:**
```
Found 47 issues, all must be fixed before merging.
```

**✅ Good:**
```
Found 3 critical security issues that must be fixed.
Found 5 important issues that should be addressed.
Also noted 12 suggestions for future improvement.

Priority: Fix the 3 critical issues first.
```

---

## 💡 Communication Tips

**Be respectful:**
```
❌ "This code is terrible"
✅ "This has a security vulnerability that needs fixing"
```

**Be specific:**
```
❌ "Error handling is wrong"
✅ "Missing try-catch on line 45 for database call, which can crash the server"
```

**Explain why:**
```
❌ "Don't do this"
✅ "Don't interpolate user input into SQL queries because it allows SQL injection attacks"
```

**Offer solutions:**
```
❌ "This is broken"
✅ "This will fail if user is null. Add optional chaining: user?.email"
```

**Acknowledge good code:**
```
✅ "Good use of TypeScript strict mode here"
✅ "Comprehensive test coverage on this module"
✅ "Clear separation of concerns in this implementation"
```

---

**Remember:** Code review improves quality and knowledge sharing. Be thorough on security and correctness, helpful on improvements, and respectful always.
