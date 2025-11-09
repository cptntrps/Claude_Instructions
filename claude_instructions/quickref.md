# Claude Instructions Quick Reference

**One-page cheat sheet for daily development**

---

## 🚀 Getting Started

```bash
# Copy to your project
cp -r claude_instructions/ /path/to/project/

# Start session
INITIATE CLAUDE CODE INSTRUCTIONS

# Quick start (experienced users)
QUICK START
```

---

## 🎛️ Commands

| Command | Purpose |
|---------|---------|
| `INITIATE CLAUDE CODE INSTRUCTIONS` | Interactive setup (recommended) |
| `QUICK START` | Fast setup, standard mode |
| `QUICK START: [MODE]` | Fast setup with specific mode |
| `ACTIVATE AGPF` | Multi-agent reasoning |
| `ACTIVATE [MODE] MODE` | Switch to specific mode |
| `SHOW SESSION STATUS` | Check current configuration |
| `RECONFIGURE SESSION` | Change settings mid-session |

---

## 🚨 Non-Negotiables (Never Break)

**Security:**
- ✋ Never commit secrets/passwords/API keys
- ✋ Always validate and sanitize inputs
- ✋ Always prevent SQL injection (parameterized queries)
- ✋ Always prevent XSS attacks

**Testing:**
- ✋ All tests must pass before commit
- ✋ Maintain ≥70% test coverage (80% for critical)
- ✋ Create tests for new features

**Quality:**
- ✋ Zero linting errors
- ✋ Pass type checking
- ✋ Build must complete

**Git:**
- ✋ Never commit to main/master
- ✋ Use conventional commits (feat:, fix:, docs:, etc.)
- ✋ Atomic commits (one logical change)

---

## 📋 Workflow Decision Tree

```
What are you doing?

Bug reported/tests failing → workflows/bug-fix.md
New feature/enhancement → workflows/feature-development.md
Code needs improvement → workflows/refactoring.md
Reviewing code → workflows/code-review.md
Need test coverage → workflows/testing.md
Preparing for release → workflows/deployment.md
```

---

## 🎯 Autonomy Levels

**✅ HIGH (Act immediately):**
- Fix linting/formatting
- Remove console.logs
- Add missing tests
- Update docs
- Fix typos
- Add types

**🤔 MEDIUM (Recommend first):**
- Implement features
- Refactor code
- Performance optimizations
- Add validation
- Minor dependency updates

**🛑 LOW (Always ask):**
- Architecture changes
- Database schema
- New major dependencies
- Breaking API changes
- Security implementations
- Multi-module changes

---

## 📝 Commit Message Format

```bash
# Format
<type>(<scope>): <description>

# Types
feat:     New feature
fix:      Bug fix
docs:     Documentation only
refactor: Code change (no behavior change)
test:     Test changes
chore:    Maintenance (deps, config, etc.)
perf:     Performance improvement
style:    Formatting (no code change)

# Examples
feat(auth): add social login with Google
fix(api): handle null user in /profile endpoint
docs(readme): update installation instructions
refactor(utils): extract duplicate validation logic
test(auth): add integration tests for login flow
chore(deps): upgrade next to 14.0.0
```

---

## 🎛️ Meta Modes

| Mode | Use When | Behavior |
|------|----------|----------|
| **STANDARD** | Normal work | Balanced (default) |
| **SPEED** | Time-critical | Minimal output, max efficiency |
| **REVIEW** | Critical systems | Ask before all changes |
| **LEARNING** | Training | Explains WHY, not just WHAT |
| **DEBUG** | Troubleshooting | Shows reasoning |
| **EVALUATION** | Testing system | Detailed citations |
| **PROTOTYPE** | Exploration | Relaxed quality (50% coverage OK) |

```
# Activate
Activate SPEED MODE

# Combine with AGPF
ACTIVATE AGPF + REVIEW MODE
```

---

## 🤖 AGPF (Advanced)

**When to use:**
- Complex multi-domain tasks
- Architectural decisions
- Need multiple expert perspectives

**Agent Roles:**
- **SME** - Domain expert (Security, Performance, Database, etc.)
- **Orchestrator** - Task coordinator

**Activation:**
```
ACTIVATE AGPF

Build a secure file upload feature with performance optimization
```

**Result:** Multiple agents analyze from different perspectives, critique approaches, and collaborate on optimal solution.

---

## 📁 File Locations

| Need | Location |
|------|----------|
| Main instructions | `claude_instructions.md` |
| Initialization | `initialization.md` |
| Meta modes | `meta-modes.md` |
| AGPF framework | `core/agpf-framework.md` |
| Workflows | `workflows/*.md` |
| Standards | `standards/*.md` |
| Project types | `project-types/*.md` |
| Advanced guides | `advanced/*.md` |
| Examples | `examples/*.md` |
| Config schema | `.claude/config.schema.json` |

---

## 🔧 Project Configuration

Create `.claude/config.json` to override defaults:

```json
{
  "projectType": "nextjs-app",
  "commands": {
    "test": "npm run test:ci",
    "lint": "npm run lint",
    "build": "npm run build",
    "typecheck": "tsc --noEmit"
  },
  "qualityGates": {
    "minCoverage": 80,
    "enforceTypes": true,
    "allowLintWarnings": false
  },
  "autonomyPreferences": {
    "autoFixLinting": true,
    "askBeforeRefactoring": false,
    "askBeforeNewDependencies": true
  }
}
```

---

## ✅ Pre-Commit Checklist

Before committing, verify:

```
□ All tests pass (npm test)
□ Linting passes (npm run lint)
□ Type checking passes (tsc --noEmit)
□ Coverage ≥70% (npm run coverage)
□ No secrets committed (check .env, keys)
□ Conventional commit message
□ Not on main/master branch
□ Code reviewed (if team)
```

---

## 🆘 Common Patterns

**Bug Fix:**
```
1. Reproduce issue
2. Write failing test
3. Fix implementation
4. Verify test passes
5. Commit: fix(scope): description
```

**New Feature:**
```
1. Understand requirements
2. Write tests (TDD)
3. Implement feature
4. Verify tests pass + coverage
5. Commit: feat(scope): description
```

**Refactoring:**
```
1. Ensure tests exist
2. Run tests (all passing)
3. Refactor code
4. Run tests again (still passing)
5. Commit: refactor(scope): description
```

---

## 🚨 CI/CD Failure Response

**Auto-fix (no ask needed):**
- Linting errors
- Formatting issues
- Simple test failures
- Missing imports
- Type errors

**Diagnose & escalate:**
- Build failures
- Complex test failures
- Flaky tests
- Infrastructure issues
- Deployment failures

---

## 📞 Need Help?

**In-session:**
- Ask Claude: "Explain [concept]"
- Check status: `SHOW SESSION STATUS`
- Switch mode: `SWITCH MODE: DEBUG MODE`

**Documentation:**
- Quick ref: This file
- Full guide: `claude_instructions.md`
- Workflows: `workflows/*.md`
- Examples: `examples/*.md`

**Feedback:**
- Report issues: https://github.com/anthropics/claude-code/issues

---

**Pro Tip:** Keep this file open in a separate tab for quick reference during development!
