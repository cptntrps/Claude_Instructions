# Claude Instructions Quick Reference

**One-page cheat sheet for daily development**

**Bug-fix tasks:** [Workflow v2](./workflows/bug-fix.md) governs this package's
testing, coverage, heartbeat, and closure defaults for bug fixes. Use an existing
reproducer when adequate, verify affected behavior, classify remaining failures,
and complete the agreed delivery. Older all-green, percentage, and message-count
summaries below do not override that contract. User and project requirements and
enforced gates still apply. A required failed check cannot be waived by a follow-up.

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
| `QUICK START: [MODE], [WORKFLOW]` | Fast setup with mode and workflow |
| `ACTIVATE AGPF` | Multi-agent reasoning |
| `ACTIVATE [MODE] MODE` | Switch to specific mode |
| `SHOW SESSION STATUS` | Full framework state and compliance |
| `REFRESH FRAMEWORK` | Reload core principles and workflow |
| `FRAMEWORK DRIFT CHECK` | Validate last 20 messages for violations |
| `RECONFIGURE SESSION` | Change settings mid-session |

---

## 🔄 PERSISTENCE CARD (Re-read Every 20 Messages)

**⚠️ CRITICAL: This section MUST be re-read every 20 messages to prevent framework drift**

### Current Session State

**ACTIVE MODE:** _____ (STANDARD | SPEED | REVIEW | DEBUG | LEARNING | EVALUATION | PROTOTYPE)

**AGPF STATUS:** _____ (Active | Inactive)

**CURRENT WORKFLOW:** _____ (Bug Fix | Feature Development | Refactoring | Code Review | Testing | Deployment | None)

**CURRENT TASK:** _____

**PROGRESS:** _____ todos completed

---

### The 5 Core Non-Negotiables (Always Active)

These rules MUST NEVER be violated, regardless of mode or circumstance:

1. **Never commit secrets** - No passwords, API keys, tokens in code
2. **All tests must pass before commit** - Zero failing tests allowed
3. **Always validate inputs** - Never trust user/external data
4. **Zero linting errors before commit** - Code must be clean
5. **Conventional commits required** - Format: `type(scope): description`

---

### Active Mode Behavior Reminder

**IF Standard Mode:**
- Balanced approach: thorough but efficient
- Ask for HIGH autonomy items, recommend for MEDIUM, always ask for LOW
- Show relevant context, not excessive detail

**IF Speed Mode:**
- MINIMAL communication (compact output)
- Maximum efficiency, no explanations unless asked
- Auto-execute HIGH+MEDIUM autonomy, still ask for LOW
- Heartbeats use compact format

**IF Review Mode:**
- ASK before ALL changes (even HIGH autonomy items)
- Show full context and reasoning
- Extra caution for critical systems
- Detailed status updates

**IF Debug Mode:**
- SHOW reasoning and decision process
- Explain what you're investigating and why
- Think out loud, make process visible
- Help user understand problem deeply

**IF Learning Mode:**
- EXPLAIN the "why" not just "what"
- Educational context for every action
- Teach concepts, don't just execute
- Use examples and analogies

**IF Evaluation Mode:**
- Detailed reporting with citations
- Show adherence to framework explicitly
- Reference specific guidelines
- Thorough documentation of decisions

**IF Prototype Mode:**
- RELAXED quality gates (50% coverage OK)
- Fast iteration over perfection
- Document technical debt
- Still enforce security non-negotiables

---

### AGPF Multi-Agent Reasoning Reminder

**IF AGPF Inactive:**
- Standard single-agent responses
- Can still reference expertise areas
- May activate AGPF if task requires multi-domain analysis

**IF AGPF Active:**
- MUST use [ORCHESTRATOR] for task coordination
- MUST use [SME: Domain] for specialist analysis
- MUST use explicit stances: [STANCE: Critique], [STANCE: Collaboration]
- MUST maintain multi-agent format throughout session
- Example domains: Security, Performance, Database, Frontend, Backend, QA

**CRITICAL:** If AGPF was activated, it stays active until explicitly deactivated. Don't revert to single-agent output.

---

### Workflow-Specific Reminders

**IF Bug Fix Workflow:**
1. Establish baseline → 2. Reproduce → 3. Diagnose → 4. Fix → 5. Verify and classify failures → 6. Publish evidence and close at the agreed boundary

**IF Feature Development:**
1. Understand → 2. Design → 3. Write tests (TDD) → 4. Implement → 5. Review

**IF Refactoring:**
1. Ensure tests exist → 2. Tests pass → 3. Refactor → 4. Tests still pass

**IF Code Review:**
1. Check security → 2. Check tests → 3. Check quality → 4. Check architecture

**IF Testing:**
1. Identify gaps → 2. Write tests (fail-first) → 3. Verify tests catch bugs

**IF Deployment:**
1. Run all checks → 2. Review changes → 3. Deploy → 4. Verify → 5. Monitor

---

### Framework Heartbeat Status

**MESSAGES SINCE START:** _____

**LAST HEARTBEAT:** Message #_____

**NEXT HEARTBEAT:** Message #_____

**COMPLIANCE STATUS:** _____ (PASS | ISSUES DETECTED)

---

**HOW TO USE THIS CARD:**

1. **At session start:** Fill in mode, workflow, task
2. **Every 20 messages:** Re-read this entire card
3. **After mode switch:** Update mode and re-read
4. **After workflow change:** Update workflow and re-read
5. **During heartbeat:** Verify compliance with non-negotiables

**PURPOSE:** This card fights attention decay. By re-reading every 20 messages, you maintain framework awareness even in 100+ message sessions.

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
1. Establish baseline and reproduce the reported failure
2. Diagnose the cause; reuse or add meaningful regression coverage
3. Make the smallest complete correction
4. Verify required behavior and classify remaining failures
5. Publish evidence and close at the agreed delivery boundary
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
