# Feature Development Workflow

Framework for implementing new features and enhancements.

---

## 🎯 Core Principle

**Build incrementally, test continuously.** Features aren't "done" until they're tested, documented, and integrated.

---

## 🚨 Non-Negotiables

Before marking a feature as complete:

- ✋ **All functionality must have tests** (unit and/or integration)
- ✋ **Minimum 70% test coverage** for new code (80% for critical paths)
- ✋ **All tests must pass**
- ✋ **No linting errors**
- ✋ **Type checking passes**
- ✋ **Build succeeds**
- ✋ **Input validation implemented** for all user inputs
- ✋ **Security review passed** (injection, XSS, auth checks)

---

## 📋 Framework

### 1. Understand Requirements

**Clarify what's needed:**
- What problem does this solve?
- Who will use it?
- What are the acceptance criteria?
- What are the edge cases?
- Are there performance requirements?
- Are there security implications?

**Questions to ask:**
- Is the requirement clear and complete?
- Are there multiple valid interpretations?
- What should happen in error cases?
- Are there dependencies on other features?

**If unclear, ask:**
```
I understand you want [feature], but I need clarification on:
- [Specific question 1]
- [Specific question 2]

Could you provide more details?
```

---

### 2. Explore Existing Codebase

**Find similar patterns:**
- Look for existing similar features
- Understand current architecture
- Identify reusable components/utilities
- Check existing patterns for consistency

**Tools:**
```
Task/Explore: "How are API endpoints structured?"
Grep: Find similar features
Read: Existing implementations to follow pattern
```

**Example:**
```
Feature: Add user profile picture upload

Explore:
- How are other file uploads handled?
- What storage system is used?
- What validation is in place?
- How are images served?

Read similar features:
- avatarUpload.ts
- documentUpload.ts
- imageValidation.ts
```

---

### 3. Design the Implementation

**Break down the feature:**
- What files need to be created/modified?
- What's the data flow?
- What are the major components?
- What dependencies are needed?

**Create task breakdown:**
```
Use TodoWrite to create task list:
- Backend: Add upload endpoint
- Backend: Add validation
- Backend: Add storage integration
- Backend: Add tests
- Frontend: Create upload component
- Frontend: Add error handling
- Frontend: Add tests
- Integration testing
```

**Design considerations:**
- **Data model:** What database changes are needed?
- **API design:** RESTful? GraphQL? Endpoints?
- **Validation:** What rules apply?
- **Error handling:** What can go wrong?
- **Security:** Auth required? Input sanitization?
- **Performance:** Will it scale?

---

### 4. Implement Incrementally

**Start with core functionality:**

```
Priority order:
1. Data layer (models, database)
2. Business logic (services, utilities)
3. API layer (controllers, routes)
4. Validation and error handling
5. Frontend components
6. Integration
```

**Example - Profile Picture Upload:**

**Step 1: Backend foundation**
```typescript
// services/imageUpload.ts
export async function uploadProfileImage(
  userId: string,
  file: File
): Promise<string> {
  // Core upload logic
  validateImageFile(file);
  const url = await storage.upload(file, `profiles/${userId}`);
  await db.users.update(userId, { profileImageUrl: url });
  return url;
}
```

**Step 2: Validation**
```typescript
function validateImageFile(file: File) {
  const MAX_SIZE = 5 * 1024 * 1024; // 5MB
  const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp'];

  if (file.size > MAX_SIZE) {
    throw new ValidationError('File too large (max 5MB)');
  }

  if (!ALLOWED_TYPES.includes(file.type)) {
    throw new ValidationError('Invalid file type');
  }
}
```

**Step 3: API endpoint**
```typescript
// routes/profile.ts
router.post('/profile/image',
  authenticate,
  upload.single('image'),
  async (req, res) => {
    try {
      const url = await uploadProfileImage(req.user.id, req.file);
      res.json({ imageUrl: url });
    } catch (error) {
      handleError(error, res);
    }
  }
);
```

**Step 4: Tests**
```typescript
describe('Profile Image Upload', () => {
  it('should upload valid image', async () => {
    const file = createMockImage({ type: 'image/jpeg', size: 1024 });
    const url = await uploadProfileImage('user-123', file);
    expect(url).toMatch(/^https:\/\//);
  });

  it('should reject oversized image', async () => {
    const file = createMockImage({ size: 10 * 1024 * 1024 });
    await expect(uploadProfileImage('user-123', file))
      .rejects
      .toThrow('File too large');
  });
});
```

**Test after each step:**
```bash
npm test -- imageUpload.test.ts
```

---

### 5. Handle Security

**Security checklist for new features:**

**Authentication:**
- [ ] Endpoint requires authentication if needed
- [ ] User can only access/modify their own data
- [ ] Role-based access control if needed

**Input Validation:**
- [ ] All inputs validated (type, format, length, range)
- [ ] File uploads: type, size, content validation
- [ ] SQL injection prevented (use parameterized queries)
- [ ] NoSQL injection prevented (validate object structure)

**Output Handling:**
- [ ] XSS prevented (escape output, use framework protections)
- [ ] Sensitive data not exposed in responses
- [ ] Errors don't leak internal info

**Example:**
```typescript
// ❌ Bad: No validation
router.post('/profile', async (req, res) => {
  await db.users.update(req.user.id, req.body);
  res.json({ success: true });
});

// ✅ Good: Validated input
router.post('/profile', async (req, res) => {
  const schema = z.object({
    name: z.string().min(1).max(100),
    bio: z.string().max(500).optional(),
    website: z.string().url().optional(),
  });

  const data = schema.parse(req.body);
  await db.users.update(req.user.id, data);
  res.json({ success: true });
});
```

**See:** [Security Checklist](../standards/security-checklist.md)

---

### 6. Write Comprehensive Tests

**Test coverage requirements:**
- **Unit tests:** Business logic, utilities, helpers
- **Integration tests:** API endpoints, database operations
- **Edge cases:** Invalid inputs, boundary conditions, errors

**Testing pyramid:**
```
      / E2E \         ← Few (critical user flows)
     /-------\
    / Integr. \       ← Some (API, database)
   /-----------\
  /    Unit     \     ← Many (functions, logic)
 /---------------\
```

**Example test suite:**
```typescript
describe('Profile Image Upload Feature', () => {
  // Happy path
  it('should upload and save profile image');

  // Validation
  it('should reject files over 5MB');
  it('should reject non-image files');
  it('should reject images with invalid dimensions');

  // Security
  it('should require authentication');
  it('should prevent uploading to other users profiles');

  // Error handling
  it('should handle storage service errors');
  it('should handle database errors');

  // Edge cases
  it('should replace existing profile image');
  it('should handle concurrent uploads');
});
```

**Run tests frequently:**
```bash
# After each implementation step
npm test

# Check coverage
npm run test:coverage
```

---

### 7. Integration

**Connect all pieces:**
- Backend API working
- Frontend component working
- End-to-end flow tested
- Error states handled
- Loading states handled

**Frontend integration example:**
```typescript
function ProfileImageUpload() {
  const [uploading, setUploading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function handleUpload(file: File) {
    setUploading(true);
    setError(null);

    try {
      const formData = new FormData();
      formData.append('image', file);

      const response = await fetch('/api/profile/image', {
        method: 'POST',
        body: formData,
      });

      if (!response.ok) {
        throw new Error(await response.text());
      }

      const { imageUrl } = await response.json();
      // Update UI with new image
    } catch (err) {
      setError(err.message);
    } finally {
      setUploading(false);
    }
  }

  return (
    // Component JSX with loading/error states
  );
}
```

---

### 8. Documentation

**Update relevant documentation:**

**Code documentation:**
```typescript
/**
 * Uploads a profile image for the authenticated user.
 *
 * @param userId - The user's unique identifier
 * @param file - The image file (JPEG, PNG, or WebP, max 5MB)
 * @returns The URL of the uploaded image
 * @throws {ValidationError} If file is invalid
 * @throws {StorageError} If upload fails
 */
export async function uploadProfileImage(
  userId: string,
  file: File
): Promise<string>
```

**API documentation:**
```markdown
## POST /api/profile/image

Upload a profile picture.

**Authentication:** Required

**Request:**
- Content-Type: multipart/form-data
- Body: { image: File }

**Validation:**
- Max file size: 5MB
- Allowed types: JPEG, PNG, WebP

**Response:**
- 200: { imageUrl: string }
- 400: { error: string } (validation error)
- 401: Unauthorized
- 500: Server error
```

**User-facing documentation (if needed):**
- How to use the feature
- Limitations and requirements
- Troubleshooting common issues

---

### 9. Review and Polish

**Before marking complete:**

**Code quality:**
- [ ] No linting errors or warnings
- [ ] TypeScript strict mode passes
- [ ] No console.logs or debug code
- [ ] Clean, readable code
- [ ] Follows project conventions

**Functionality:**
- [ ] All acceptance criteria met
- [ ] Edge cases handled
- [ ] Error messages are helpful
- [ ] Loading states implemented
- [ ] Success feedback provided

**Performance:**
- [ ] No obvious performance issues
- [ ] Queries optimized (no N+1)
- [ ] Large lists paginated
- [ ] Images optimized

**Accessibility (frontend):**
- [ ] Keyboard navigation works
- [ ] Screen reader friendly
- [ ] Color contrast sufficient
- [ ] Focus indicators visible

---

### 10. Commit and Deploy

**Commit strategy:**

**Option A - Single commit (small feature):**
```bash
git add .
git commit -m "$(cat <<'EOF'
feat(profile): add profile image upload

- Upload endpoint with validation
- Frontend upload component
- Image size/type validation
- Tests for happy path and edge cases

Closes #789
EOF
)"
```

**Option B - Multiple commits (larger feature):**
```bash
# Commit 1
git add services/imageUpload.ts tests/imageUpload.test.ts
git commit -m "feat(profile): add image upload service"

# Commit 2
git add routes/profile.ts
git commit -m "feat(profile): add image upload API endpoint"

# Commit 3
git add components/ProfileImageUpload.tsx
git commit -m "feat(profile): add image upload UI component"
```

**Push and create PR (if required):**
```bash
git push -u origin feature/profile-image-upload
gh pr create --title "Add profile image upload" --body "$(cat <<'EOF'
## Summary
- Adds profile image upload functionality
- Validates file type and size
- Integrates with existing storage service

## Test Plan
- [ ] Upload valid image
- [ ] Verify file size limit (5MB)
- [ ] Verify file type restriction
- [ ] Test error handling
- [ ] Test UI loading/error states
EOF
)"
```

---

## 🎚️ Adaptive Approach

### Small Feature (< 100 lines)

**Streamlined:**
```
1. Understand requirement
2. Find similar code
3. Implement + tests together
4. Verify tests pass
5. Commit
```

---

### Medium Feature (100-500 lines)

**Structured:**
```
1. Understand + clarify
2. Create task breakdown (TodoWrite)
3. Implement incrementally
4. Test each piece
5. Integration test
6. Document
7. Commit
```

---

### Large Feature (> 500 lines)

**Phased:**
```
1. Full design discussion with user
2. Break into phases/milestones
3. Implement Phase 1 completely
4. Deploy and validate
5. Continue with Phase 2
6. Consider feature flags
```

---

## ⚠️ Common Pitfalls

### Skipping Tests

**❌ Bad:**
```
Feature works manually, ship it!
```

**✅ Good:**
```
Feature works + 15 automated tests covering all cases
```

---

### Overengineering

**❌ Bad:**
```
Build flexible, configurable system that handles every
possible future requirement...
```

**✅ Good:**
```
Build exactly what's needed now.
Refactor later when requirements change.
```

---

### Poor Error Handling

**❌ Bad:**
```javascript
try {
  await uploadImage(file);
} catch (error) {
  console.log(error);
}
```

**✅ Good:**
```javascript
try {
  await uploadImage(file);
} catch (error) {
  logger.error('Image upload failed', { userId, error });

  if (error instanceof ValidationError) {
    throw new ApiError(400, error.message);
  }

  throw new ApiError(500, 'Upload failed. Please try again.');
}
```

---

## 📊 Checklist

Feature completion checklist:

- [ ] Requirements fully understood
- [ ] Similar patterns identified and followed
- [ ] Task breakdown created (for complex features)
- [ ] Core functionality implemented
- [ ] Input validation added
- [ ] Error handling implemented
- [ ] Security review passed
- [ ] Unit tests written
- [ ] Integration tests written
- [ ] All tests passing (100%)
- [ ] Test coverage ≥ 70% (80% for critical)
- [ ] No linting errors
- [ ] Type checking passes
- [ ] Build succeeds
- [ ] Code documented
- [ ] API documented (if applicable)
- [ ] Committed with proper message
- [ ] Ready for review/deployment

---

**Remember:** Features aren't done until they're tested, secure, documented, and delivered. Build incrementally and verify continuously.
