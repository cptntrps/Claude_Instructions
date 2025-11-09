# Deployment Workflow

Framework for CI/CD, releases, and deployment preparation.

---

## 🎯 Core Principle

**Deploy with confidence.** Every deployment should be tested, verified, and reversible.

---

## 🚨 Non-Negotiables

Before any deployment:

- ✋ **All tests must pass**
- ✋ **Build must succeed**
- ✋ **No linting errors**
- ✋ **Type checking passes**
- ✋ **No unhandled security vulnerabilities**
- ✋ **Environment variables configured**
- ✋ **Database migrations tested** (if applicable)

---

## 📋 Pre-Deployment Checklist

### Code Quality
- [ ] All tests passing (`npm test`)
- [ ] Linting passes (`npm run lint`)
- [ ] Type checking passes (`npm run typecheck` or `tsc --noEmit`)
- [ ] Build succeeds (`npm run build`)
- [ ] No console.logs or debug code
- [ ] Test coverage ≥ 70%

### Security
- [ ] No secrets in code
- [ ] Dependencies scanned (`npm audit` or `npm audit --audit-level=high`)
- [ ] Environment variables documented
- [ ] Authentication/authorization working
- [ ] HTTPS configured (production)

### Database
- [ ] Migrations tested locally
- [ ] Migrations have rollback (DOWN migration)
- [ ] Backup taken (production)
- [ ] Seed data prepared (if needed)

### Configuration
- [ ] Environment variables set
- [ ] API keys configured
- [ ] Database connection string correct
- [ ] Feature flags set appropriately
- [ ] CORS configured correctly

### Documentation
- [ ] CHANGELOG updated
- [ ] Version bumped (semantic versioning)
- [ ] Deployment notes prepared
- [ ] Rollback procedure documented

---

## 📋 Framework

### 1. Pre-Commit Checks

**Before committing:**

```bash
# Run all quality checks in parallel
npm test & \
npm run lint & \
npm run typecheck & \
wait

# If all pass, commit
git commit -m "feat: add new feature"
```

**Automated pre-commit hooks:**
```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "npm run lint && npm test",
      "pre-push": "npm test && npm run build"
    }
  }
}
```

---

### 2. Continuous Integration (CI)

**Typical CI pipeline:**

```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm test
      - run: npm run build

      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

**When CI fails:**
- Read the error logs
- Identify the failing step
- Reproduce locally if needed
- Fix the issue
- Commit and push fix
- Verify CI passes

See: [CI/CD Failure Handling](../claude_instructions.md#ci-cd-failure-handling)

---

### 3. Version Management

**Semantic Versioning (MAJOR.MINOR.PATCH):**
- **MAJOR:** Breaking changes (v2.0.0)
- **MINOR:** New features, backward compatible (v1.1.0)
- **PATCH:** Bug fixes (v1.0.1)

**Bumping version:**
```bash
# Patch (bug fix)
npm version patch  # 1.0.0 → 1.0.1

# Minor (new feature)
npm version minor  # 1.0.1 → 1.1.0

# Major (breaking change)
npm version major  # 1.1.0 → 2.0.0
```

**Update CHANGELOG:**
```markdown
# Changelog

## [1.2.0] - 2024-11-09

### Added
- Profile image upload feature
- Email verification flow

### Fixed
- Login bug with unverified emails (#456)

### Changed
- Updated dependencies to fix security vulnerabilities
```

---

### 4. Database Migrations

**Before deploying with database changes:**

**Test migrations locally:**
```bash
# Run migration
npm run migrate:up

# Verify database structure
npm run db:inspect

# Test rollback
npm run migrate:down

# Re-run migration
npm run migrate:up
```

**Migration checklist:**
- [ ] Migration creates expected schema changes
- [ ] DOWN migration reverts changes completely
- [ ] Existing data preserved or migrated appropriately
- [ ] No data loss
- [ ] Migration is idempotent (can run multiple times)

**Example migration:**
```typescript
// migrations/20241109_add_email_verified.ts
export async function up(db: Knex) {
  await db.schema.table('users', (table) => {
    table.boolean('email_verified').defaultTo(false);
    table.timestamp('email_verified_at').nullable();
  });

  // Backfill existing users as verified
  await db('users').update({ email_verified: true });
}

export async function down(db: Knex) {
  await db.schema.table('users', (table) => {
    table.dropColumn('email_verified');
    table.dropColumn('email_verified_at');
  });
}
```

**Production migration:**
```bash
# Take backup first!
npm run db:backup

# Run migration
npm run migrate:up

# Verify success
npm run db:inspect

# If failed, rollback
npm run migrate:down
npm run db:restore
```

---

### 5. Environment Configuration

**Environment variables:**

**Development (.env.development):**
```bash
NODE_ENV=development
DATABASE_URL=postgresql://localhost/myapp_dev
API_KEY=dev_key_12345
DEBUG=true
```

**Production (.env.production - DO NOT COMMIT):**
```bash
NODE_ENV=production
DATABASE_URL=postgresql://prod-server/myapp
API_KEY=<from-secrets-manager>
DEBUG=false
```

**Document required variables (.env.example):**
```bash
NODE_ENV=development
DATABASE_URL=postgresql://localhost/myapp
API_KEY=your_api_key_here
EMAIL_SERVICE_URL=https://api.emailservice.com
```

**Verify environment:**
```bash
# Check all required variables are set
npm run check:env

# Or manually
node -e "console.log(process.env.DATABASE_URL)"
```

---

### 6. Deployment Process

#### Option A: Manual Deployment

```bash
# 1. Pull latest code
git pull origin main

# 2. Install dependencies
npm ci

# 3. Run migrations
npm run migrate:up

# 4. Build
npm run build

# 5. Restart application
pm2 restart app
```

---

#### Option B: Automated Deployment (CI/CD)

**Example: Deploy to Vercel/Netlify**
```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
```

---

#### Option C: Docker Deployment

**Dockerfile:**
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

EXPOSE 3000

CMD ["npm", "start"]
```

**Deploy:**
```bash
# Build image
docker build -t myapp:1.2.0 .

# Run locally to test
docker run -p 3000:3000 --env-file .env.production myapp:1.2.0

# Push to registry
docker push myregistry.com/myapp:1.2.0

# Deploy to production
kubectl set image deployment/myapp myapp=myregistry.com/myapp:1.2.0
```

---

### 7. Post-Deployment Verification

**Immediately after deployment:**

```bash
# 1. Check application is running
curl https://myapp.com/health

# 2. Check logs for errors
npm run logs:prod
# or
kubectl logs deployment/myapp

# 3. Test critical endpoints
curl https://myapp.com/api/users/me

# 4. Monitor error rates
# Check monitoring dashboard (DataDog, Sentry, etc.)
```

**Smoke tests:**
```bash
# Run automated smoke tests
npm run test:smoke

# Or manual critical path:
# - Can users log in?
# - Can users access main features?
# - Are API endpoints responding?
# - Is database accessible?
```

---

### 8. Rollback Procedure

**If deployment causes issues:**

**Option A: Revert Code**
```bash
# Find previous working commit
git log --oneline

# Revert to previous version
git revert HEAD

# Or reset if not pushed
git reset --hard <previous-commit-hash>

# Redeploy
npm run deploy
```

**Option B: Redeploy Previous Version**
```bash
# If using versioned deployments
npm run deploy --version=1.1.0

# Or with Docker
kubectl set image deployment/myapp myapp=myregistry.com/myapp:1.1.0
```

**Option C: Rollback Database Migration**
```bash
# Rollback migration
npm run migrate:down

# Restore database backup if needed
npm run db:restore
```

See: [Rollback & Recovery](../advanced/rollback-recovery.md)

---

## 🎚️ Deployment Strategies

### Blue-Green Deployment
- Deploy new version (green) alongside old version (blue)
- Switch traffic to green when verified
- Keep blue running for quick rollback

### Canary Deployment
- Deploy to small percentage of users first
- Monitor metrics
- Gradually increase percentage
- Full rollout if metrics look good

### Rolling Deployment
- Update instances one at a time
- Maintain service availability
- Rollback if issues detected

---

## ⚠️ Common Issues

### Environment Variables Not Set

**Symptom:** App crashes with "undefined is not a function" or "Cannot read property of undefined"

**Solution:**
```bash
# Verify all required variables
node -e "const required = ['DATABASE_URL', 'API_KEY']; required.forEach(v => { if (!process.env[v]) console.error('Missing:', v) })"
```

---

### Database Migration Failed

**Symptom:** Migration runs but schema is wrong

**Solution:**
```bash
# Rollback migration
npm run migrate:down

# Fix migration file
# Test locally
npm run migrate:up

# Redeploy
```

---

### Build Fails in CI But Works Locally

**Symptom:** `npm run build` works locally but fails in CI

**Common causes:**
- Different Node.js versions
- Missing environment variables in CI
- Platform-specific dependencies

**Solution:**
```bash
# Match Node version in CI config
# Set environment variables in CI settings
# Check CI logs for specific error
```

---

## 📊 Deployment Checklist

### Pre-Deployment
- [ ] All tests passing
- [ ] Build succeeds
- [ ] Linting passes
- [ ] Type checking passes
- [ ] Security audit clean
- [ ] Migrations tested
- [ ] Environment variables set
- [ ] Version bumped
- [ ] CHANGELOG updated
- [ ] Backup taken (production database)

### Deployment
- [ ] Code deployed
- [ ] Migrations run
- [ ] Application started
- [ ] Health check passes

### Post-Deployment
- [ ] Smoke tests passed
- [ ] Critical paths verified
- [ ] Logs checked for errors
- [ ] Monitoring dashboard reviewed
- [ ] Performance metrics normal
- [ ] Error rates normal

### Rollback Ready
- [ ] Previous version available
- [ ] Rollback procedure documented
- [ ] Database backup available
- [ ] Team notified of deployment

---

## 💡 Example: Complete Deployment

```bash
# 1. Pre-deployment checks
npm test && npm run lint && npm run typecheck && npm run build

# 2. Bump version and update changelog
npm version minor
# Edit CHANGELOG.md
git add CHANGELOG.md
git commit --amend --no-edit

# 3. Tag release
git tag v1.2.0
git push origin main --tags

# 4. CI/CD automatically:
#    - Runs tests
#    - Builds application
#    - Deploys to staging
#    - Runs smoke tests
#    - Deploys to production

# 5. Post-deployment verification
curl https://myapp.com/health
npm run test:smoke:prod

# 6. Monitor for 15 minutes
# Check error rates, response times, logs

# 7. If issues found, rollback:
npm run deploy --version=1.1.0
```

---

**Remember:** Good deployments are boring. Thorough preparation and automation make deployments safe and routine.
