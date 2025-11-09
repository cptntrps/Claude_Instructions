# Claude Development Instructions

**Version:** 1.0.0
**Last Updated:** 2025-11-09

A comprehensive, reusable instruction system for Claude Code that defines agentic workflows, best practices, and quality standards for software development projects.

---

## 🎯 Purpose

This repository contains a modular instruction system that guides Claude through various development scenarios with:

- **Principle-oriented workflows** with non-negotiable safety guardrails
- **Adaptive autonomy** based on task risk and complexity
- **Comprehensive quality standards** for security, testing, and code quality
- **Project-specific configurations** for different tech stacks
- **Advanced guides** for complex scenarios (rollback, migrations, security)

---

## 🚀 Quick Start

### For Each New Claude Code Session

Point Claude to the main instruction file:

```
Read and follow the instructions in claude_instructions.md
```

That's it! Claude will:
1. Load the core principles and non-negotiable rules
2. Auto-detect your project type (Next.js, Express, React Native, etc.)
3. Apply appropriate workflows and standards
4. Adapt autonomy based on task complexity

---

## 📁 Structure

```
Claude_Instructions/
├── claude_instructions.md          # Main entry point - START HERE
│
├── core/                            # Fundamental principles
│   ├── principles.md                # Core development principles
│   ├── tool-usage-guide.md         # How to use Claude tools effectively
│   └── communication-standards.md  # How Claude should communicate
│
├── workflows/                       # Task-specific workflows
│   ├── bug-fix.md                  # Investigating and fixing bugs
│   ├── feature-development.md      # Building new features
│   ├── code-review.md              # Reviewing code quality
│   ├── refactoring.md              # Improving code structure
│   ├── testing.md                  # Writing and maintaining tests
│   └── deployment.md               # CI/CD and deployment preparation
│
├── standards/                       # Quality standards
│   ├── git-conventions.md          # Commit messages, branches, PRs
│   ├── testing-standards.md        # Coverage requirements, test types
│   ├── security-checklist.md       # OWASP, secrets, validation
│   └── code-quality.md             # Linting, complexity, naming
│
├── project-types/                   # Project-specific configurations
│   ├── nextjs-webapp.md            # Next.js applications
│   ├── express-api.md              # Express.js APIs
│   ├── react-native-mobile.md      # React Native mobile apps
│   ├── flutter-mobile.md           # Flutter mobile apps
│   └── fastapi-python.md           # FastAPI Python APIs
│
├── advanced/                        # Advanced feature guides
│   ├── rollback-recovery.md        # Handling deployment failures
│   ├── secrets-management.md       # Secure credential handling
│   ├── database-migrations.md      # Safe schema changes
│   ├── dependency-management.md    # Package updates, security
│   └── data-validation.md          # Input validation patterns
│
└── .claude/
    └── config.schema.json          # JSON schema for project configs
```

---

## 🎚️ Customization

### Project-Specific Override

Create `.claude/config.json` in your project to override defaults:

```json
{
  "projectType": "nextjs-webapp",
  "commands": {
    "test": "npm run test:ci",
    "lint": "npm run lint -- --max-warnings 0",
    "build": "npm run build"
  },
  "qualityGates": {
    "minCoverage": 80,
    "enforceTypes": true,
    "enforceNoWarnings": false
  },
  "autonomyPreferences": {
    "autoFixLinting": true,
    "askBeforeRefactoring": false,
    "askBeforeNewDependencies": true
  },
  "customInstructions": "Always check our internal API design guide at docs/api-patterns.md before implementing endpoints."
}
```

See [`.claude/config.schema.json`](.claude/config.schema.json) for all available options.

---

## 🎛️ Meta Modes

**NEW in v1.1.0:** Activate different operational modes for specific contexts.

### Available Modes

**🔍 EVALUATION MODE** - For testing the instruction system itself
```
Activate EVALUATION MODE.
```
- Provides detailed reporting with citations
- Explains which workflow/principle is being followed
- Reports on non-negotiable compliance
- Perfect for validating the instruction system

**⚡ SPEED MODE** - For maximum efficiency
```
Activate SPEED MODE.
```
- Minimal communication, maximum action
- Still enforces all non-negotiables
- Perfect for routine tasks and production work

**🔒 REVIEW MODE** - For critical systems
```
Activate REVIEW MODE.
```
- Extra caution, asks before ALL changes
- Shows full diffs and impact analysis
- Perfect for production databases, financial systems

**📚 LEARNING MODE** - For education and mentoring
```
Activate LEARNING MODE.
```
- Explains WHY, not just WHAT
- References documentation and best practices
- Perfect for training and knowledge building

**🐛 DEBUG MODE** - For troubleshooting
```
Activate DEBUG MODE.
```
- Shows reasoning and decision-making
- Explains tool choices and alternatives
- Perfect for understanding Claude's behavior

**🚀 PROTOTYPE MODE** - For fast exploration
```
Activate PROTOTYPE MODE.
```
- Relaxed quality gates (50% coverage OK)
- Focus on working code over perfect code
- Still enforces security rules
- Perfect for POCs and spike work

**See [meta-modes.md](meta-modes.md) for complete documentation.**

---

## 🔑 Key Features

### 1. Principle-Oriented with Guardrails

**Flexible workflows guided by principles:**
- Understand before acting
- Test-driven mindset
- Security by default
- Minimal scope changes
- Fail gracefully

**But with non-negotiable safety rules:**
- Never commit secrets
- Always validate inputs
- Always pass tests before committing
- Zero linting errors
- Conventional commits required

### 2. Adaptive Autonomy

Claude adapts decision-making based on risk:

**High Autonomy** (just do it):
- Fix linting errors
- Add missing tests
- Update documentation

**Medium Autonomy** (recommend & proceed):
- Implement features
- Refactor code
- Optimize performance

**Low Autonomy** (always ask):
- Architecture changes
- Database schema changes
- Breaking API changes

### 3. Comprehensive Standards

**Security:** OWASP top 10, input validation, secrets management
**Testing:** 70%+ coverage, unit/integration/e2e patterns
**Code Quality:** Linting, type checking, complexity limits
**Git:** Conventional commits, atomic changes, PR templates

### 4. Project Type Auto-Detection

Automatically detects and adapts to:
- Next.js (detects `next.config.js`)
- Express (detects `express` in package.json)
- React Native (detects `react-native`)
- Flutter (detects `pubspec.yaml`)
- FastAPI (detects `fastapi` in requirements)

Each project type has specific commands, patterns, and best practices.

---

## 📖 Usage Examples

### Example 1: Bug Fix

```
User: "The login endpoint is returning 500 errors"

Claude:
1. Reads bug-fix.md workflow
2. Investigates: reads login code, checks logs
3. Identifies issue: missing null check
4. Creates failing test
5. Implements fix
6. Verifies test passes
7. Runs full suite
8. Commits: "fix(auth): add null check in login endpoint"
```

### Example 2: New Feature

```
User: "Add profile image upload"

Claude:
1. Reads feature-development.md workflow
2. Explores existing upload patterns
3. Creates task breakdown
4. Implements: validation, storage, API, tests
5. Security check: file type validation, size limits
6. Runs all quality gates
7. Commits: "feat(profile): add profile image upload"
```

### Example 3: Deployment Issue

```
User: "Production is down after deployment!"

Claude:
1. Reads rollback-recovery.md
2. Assesses severity: Critical
3. Executes rollback procedure
4. Verifies previous version restored
5. Investigates issue offline
6. Provides post-mortem analysis
```

---

## ✅ Validation Checklist

Use this checklist to validate the instruction system is working correctly:

### Setup Validation
- [ ] Repository structure matches expected layout
- [ ] All markdown files are present and readable
- [ ] `.claude/config.schema.json` is valid JSON

### Workflow Validation
- [ ] Test bug fix scenario: Claude creates test, fixes, verifies
- [ ] Test feature scenario: Claude breaks down task, implements incrementally
- [ ] Test refactoring scenario: Claude maintains behavior, tests pass

### Standards Validation
- [ ] Claude rejects commits with secrets
- [ ] Claude enforces test coverage minimums
- [ ] Claude uses conventional commit messages
- [ ] Claude runs linting before committing

### Autonomy Validation
- [ ] Claude auto-fixes linting without asking
- [ ] Claude asks before adding new dependencies
- [ ] Claude asks before database schema changes
- [ ] Claude explains decisions for refactoring

### Project Type Validation
- [ ] Correct auto-detection for your project type
- [ ] Appropriate commands used (npm vs pip vs flutter)
- [ ] Project-specific patterns followed

---

## 🔄 Maintenance

### Updating Instructions

1. Edit the relevant markdown files
2. Update version number in main `claude_instructions.md`
3. Document changes in this README
4. Test with actual Claude Code session

### Adding New Project Types

1. Create file in `project-types/` directory
2. Include: detection criteria, commands, patterns
3. Add to main `claude_instructions.md` detection table
4. Update `.claude/config.schema.json` enum

### Adding New Workflows

1. Create file in `workflows/` directory
2. Follow existing structure: Core Principle, Non-Negotiables, Framework
3. Add to main `claude_instructions.md` workflow table

---

## 🤝 Contributing

This is an internal instruction system. To propose changes:

1. Test changes in actual development scenarios
2. Ensure backward compatibility
3. Document rationale for changes
4. Update version number and changelog

---

## 📊 Approach Summary

| Aspect | Approach |
|--------|----------|
| **Workflow Depth** | Principle-oriented with non-negotiable rules |
| **Quality Gates** | 70% coverage, zero linting errors, type checking, security validation |
| **Autonomy** | Adaptive: High for safe tasks, Low for risky changes |
| **CI/CD Failures** | Auto-fix simple issues, escalate complex ones |
| **Project Integration** | Hybrid: Auto-detect with manual override |
| **Structure** | Modular: Core + Workflows + Standards + Project Types + Advanced |

---

## 🎓 Philosophy

**"Principle-oriented with guardrails"**

We give Claude maximum flexibility to apply engineering judgment and adapt to situations, but we enforce non-negotiable safety rules that protect security, quality, and correctness.

Think of it like giving a senior engineer autonomy, but with clear policies they must always follow (no secrets in code, tests must pass, etc.).

---

## 📝 Changelog

### v1.1.0 (2025-11-09)
- **NEW:** Meta Modes system for different operational contexts
  - EVALUATION MODE: Test/validate instruction system with detailed reporting
  - DEBUG MODE: Show reasoning and decision-making process
  - LEARNING MODE: Educational explanations for mentoring
  - SPEED MODE: Maximum efficiency for production work
  - REVIEW MODE: Extra caution for critical systems
  - PROTOTYPE MODE: Fast iteration with relaxed quality gates
- Added comprehensive meta-modes.md documentation
- Updated main instruction file to reference meta modes

### v1.0.0 (2025-11-09)
- Initial release
- Core principles and workflows
- Project type detection for Next.js, Express, React Native, Flutter, FastAPI
- Advanced guides: rollback, secrets, migrations, dependencies, validation
- Adaptive autonomy system
- Quality standards and security checklists

---

## 📞 Support

For questions or issues with this instruction system, refer to specific sections:

- Workflow issues → `workflows/`
- Quality standards → `standards/`
- Project-specific → `project-types/`
- Advanced scenarios → `advanced/`

---

**Version:** 1.0.0
**License:** Internal Use
**Maintained By:** Development Team
