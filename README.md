# Claude Development Instructions

**Version:** 1.3.0
**Last Updated:** 2025-11-09

A comprehensive, reusable instruction system for Claude Code that defines agentic workflows, best practices, and quality standards for software development projects.

---

## 🎯 Purpose

This repository contains a **portable instruction system** in the `claude_instructions/` folder that you can copy to any project. Claude will then follow these instructions to provide consistent, high-quality development assistance across all your projects.

**Key Features:**
- **Principle-oriented workflows** with non-negotiable safety guardrails
- **Adaptive autonomy** based on task risk and complexity
- **6 operational modes** for different contexts (Speed, Review, Debug, etc.)
- **Interactive initialization** with guided setup
- **Comprehensive quality standards** for security, testing, and code quality
- **Project-specific configurations** for different tech stacks
- **Advanced guides** for complex scenarios (rollback, migrations, security)

---

## 🚀 Quick Start

### **Step 1: Copy to Your Project**

Copy the `claude_instructions/` folder into your project:

```bash
# From this repository
cp -r claude_instructions/ /path/to/your/project/

# Or clone and copy
git clone <this-repo-url>
cp -r Claude_Instructions/claude_instructions/ /path/to/your/project/
```

**Result:** Your project now has a `claude_instructions/` folder with all the guidance.

---

### **Step 2: Start Claude Code Session**

In your project, start a new Claude Code session and type:

```
INITIATE CLAUDE CODE INSTRUCTIONS
```

**That's it!** Claude will:
1. ✅ Load the instruction system from `claude_instructions/`
2. 🔍 Auto-detect your project type (Next.js, Express, React Native, etc.)
3. 🎛️ Present a menu to select operational mode
4. ✅ Validate your development environment
5. 📋 Show common tasks menu
6. 🚀 Activate the appropriate workflow

---

### **Alternative Methods**

#### **Quick Start (No Menus)**
```
QUICK START
```
Loads instructions, detects project, uses standard mode, no interaction.

#### **Quick Start with Mode**
```
QUICK START: SPEED MODE
```
Immediately activates a specific mode.

#### **Manual (Traditional)**
```
Read and follow the instructions in claude_instructions/claude_instructions.md
```
Silent loading, you direct everything.

---

## 📁 What's in the Folder

```
claude_instructions/
├── claude_instructions.md          # Main entry point - START HERE
├── initialization.md               # Interactive setup script
├── meta-modes.md                   # Meta modes documentation
│
├── core/                            # Fundamental principles
│   ├── principles.md                # Core development principles
│   ├── tool-usage-guide.md         # How to use Claude tools
│   └── communication-standards.md  # Communication guidelines
│
├── workflows/                       # Task-specific workflows
│   ├── bug-fix.md                  # Investigating and fixing bugs
│   ├── feature-development.md      # Building new features
│   ├── code-review.md              # Reviewing code quality
│   ├── refactoring.md              # Improving code structure
│   ├── testing.md                  # Writing and maintaining tests
│   └── deployment.md               # CI/CD and deployment
│
├── standards/                       # Quality standards
│   ├── git-conventions.md          # Commit messages, branches, PRs
│   ├── testing-standards.md        # Coverage requirements
│   ├── security-checklist.md       # OWASP, secrets, validation
│   └── code-quality.md             # Linting, complexity, naming
│
├── project-types/                   # Project-specific configs
│   ├── nextjs-webapp.md            # Next.js applications
│   ├── express-api.md              # Express.js APIs
│   ├── react-native-mobile.md      # React Native apps
│   ├── flutter-mobile.md           # Flutter apps
│   └── fastapi-python.md           # FastAPI Python APIs
│
├── advanced/                        # Advanced guides
│   ├── rollback-recovery.md        # Deployment failures & rollback
│   ├── secrets-management.md       # Secure credential handling
│   ├── database-migrations.md      # Safe schema changes
│   ├── dependency-management.md    # Package updates, security
│   └── data-validation.md          # Input validation patterns
│
└── .claude/
    └── config.schema.json          # JSON schema for project configs
```

**Total:** 28 files, 11,400+ lines of comprehensive guidance

---

## 🎛️ Meta Modes

The instruction system supports 6 operational modes:

| Mode | When to Use | Behavior |
|------|-------------|----------|
| 🔍 **EVALUATION** | Testing/validating instructions | Detailed reporting with citations |
| ⚡ **SPEED** | Production work, routine tasks | Minimal communication, max efficiency |
| 🔒 **REVIEW** | Critical systems, production DBs | Extra caution, asks before ALL changes |
| 📚 **LEARNING** | Training, education | Explains WHY, not just WHAT |
| 🐛 **DEBUG** | Troubleshooting | Shows reasoning and decisions |
| 🚀 **PROTOTYPE** | POCs, exploration | Relaxed quality gates, fast iteration |
| ⚙️ **STANDARD** | General development | Balanced approach (default) |

**Activate during initialization or use:**
```
SWITCH MODE: [MODE NAME]
```

**See:** [`claude_instructions/meta-modes.md`](claude_instructions/meta-modes.md) for details.

---

## 🎚️ Customization

### **Project-Specific Configuration**

Create `.claude/config.json` in your project root (not inside `claude_instructions/`):

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
  "customInstructions": "Always check our API design guide at docs/api-patterns.md"
}
```

**Schema:** See `claude_instructions/.claude/config.schema.json`

---

## 🔑 Key Features

### **1. Principle-Oriented with Guardrails**

**Flexible workflows guided by principles:**
- Understand before acting
- Test-driven mindset
- Security by default
- Minimal scope changes
- Fail gracefully

**But with non-negotiable safety rules:**
- Never commit secrets
- Always validate inputs
- All tests must pass
- Zero linting errors
- Conventional commits required

### **2. Adaptive Autonomy**

Claude adjusts decision-making based on risk:

**High Autonomy** (auto-execute):
- Fix linting errors
- Remove console.logs
- Add missing tests

**Medium Autonomy** (recommend & proceed):
- Implement features
- Refactor code
- Performance optimizations

**Low Autonomy** (always ask):
- Architecture changes
- Database schema changes
- Breaking API changes

### **3. Project Type Detection**

Auto-detects and adapts to your stack:
- Next.js Web Apps
- Express APIs
- React Native Mobile
- Flutter Mobile
- FastAPI Python

### **4. Comprehensive Workflows**

6 battle-tested workflows:
- 🐛 Bug Fix
- ✨ Feature Development
- 🔧 Refactoring
- 📝 Code Review
- 🧪 Testing
- 🚀 Deployment

### **5. Advanced Scenarios**

Handles complex situations:
- Rollback & Recovery (production incidents)
- Database Migrations (schema changes)
- Secrets Management (secure credentials)
- Dependency Management (security updates)
- Data Validation (input sanitization)

---

## 📊 Usage Examples

### **Example 1: New Project Setup**

```bash
# Create new Next.js project
npx create-next-app@latest my-app
cd my-app

# Copy instructions
cp -r ~/Claude_Instructions/claude_instructions/ .

# Start Claude Code
# In Claude:
INITIATE CLAUDE CODE INSTRUCTIONS

# Claude detects Next.js, shows menu, you select task
# Ready to develop!
```

---

### **Example 2: Existing Project**

```bash
# Add to existing project
cd my-existing-project
cp -r ~/Claude_Instructions/claude_instructions/ .

# Optional: Create project config
cat > .claude/config.json << 'EOF'
{
  "projectType": "nextjs-webapp",
  "qualityGates": {
    "minCoverage": 80
  },
  "customInstructions": "Use our company's design system in components/"
}
EOF

# Start Claude Code
INITIATE CLAUDE CODE INSTRUCTIONS
```

---

### **Example 3: Multiple Projects**

```bash
# Same instructions across all projects
for project in project1 project2 project3; do
  cp -r claude_instructions/ ~/projects/$project/
done

# Each project can have its own .claude/config.json for customization
```

---

## 🧪 Testing the System

### **Validation Checklist**

After copying to a project:

- [ ] Copy `claude_instructions/` folder to project
- [ ] Start Claude Code session
- [ ] Run `INITIATE CLAUDE CODE INSTRUCTIONS`
- [ ] Verify project type detected correctly
- [ ] Select a mode (recommend EVALUATION for first time)
- [ ] Choose a task from menu
- [ ] Verify Claude follows the workflow
- [ ] Check non-negotiable rules enforced

### **Test Scenarios**

**Test 1: Linting (High Autonomy)**
```
Create file with linting errors
Ask Claude to fix
Expected: Auto-fixes without asking
```

**Test 2: Bug Fix (Full Workflow)**
```
Report a bug
Expected: Creates test first, fixes, verifies all tests pass
```

**Test 3: Security (Non-Negotiable)**
```
Ask to add API integration
Expected: Uses environment variables, never hardcodes secrets
```

**Test 4: Database Change (Low Autonomy)**
```
Ask to add column to table
Expected: Shows migration plan, asks for approval before proceeding
```

---

## 🔄 Updating Instructions

### **When New Version Released**

```bash
# Pull latest
cd ~/Claude_Instructions
git pull

# Update all projects
for project in ~/projects/*; do
  if [ -d "$project/claude_instructions" ]; then
    echo "Updating $project"
    rm -rf "$project/claude_instructions"
    cp -r claude_instructions/ "$project/"
  fi
done
```

---

## 🛠️ Maintenance

### **Adding to .gitignore** (Optional)

If you don't want to commit instructions to project repos:

```bash
# In your project
echo "claude_instructions/" >> .gitignore
```

Then instructions stay local, not in git.

### **Committing to Project** (Recommended)

To ensure team members have same instructions:

```bash
# Commit the folder
git add claude_instructions/
git commit -m "docs: add Claude Code instructions"
```

Team members get instructions when they clone.

---

## 📖 Documentation

### **For Users:**
- **[Quick Start Guide](#-quick-start)** - Get started in 2 steps
- **[Meta Modes](claude_instructions/meta-modes.md)** - Operational modes
- **[Initialization](claude_instructions/initialization.md)** - Interactive setup

### **For Developers:**
- **[Core Principles](claude_instructions/core/principles.md)** - Engineering values
- **[Workflows](claude_instructions/workflows/)** - Task-specific guides
- **[Standards](claude_instructions/standards/)** - Quality requirements

---

## 🎓 Philosophy

**"Principle-oriented with guardrails"**

We give Claude maximum flexibility to apply engineering judgment and adapt to situations, but we enforce non-negotiable safety rules that protect security, quality, and correctness.

Think of it like giving a senior engineer autonomy, but with clear policies they must always follow (no secrets in code, tests must pass, etc.).

---

## 📝 Changelog

### v1.3.0 (2025-11-09)
- **BREAKING:** Restructured to use `claude_instructions/` folder
  - All instruction files now in portable folder
  - Easy to copy to any project
  - Updated all documentation for new structure
  - Simplified deployment and reuse
- Updated README with installation and usage instructions
- Added examples for single and multi-project setups

### v1.2.0 (2025-11-09)
- **NEW:** Interactive initialization script
  - `INITIATE CLAUDE CODE INSTRUCTIONS` command
  - Auto-detects project type and validates environment
  - Interactive mode selection and task menu
  - `QUICK START` commands for experienced users
- Added initialization.md with complete interactive flow

### v1.1.0 (2025-11-09)
- **NEW:** Meta Modes system
  - 6 operational modes (Evaluation, Speed, Review, Learning, Debug, Prototype)
  - Mode switching during session
  - Comprehensive meta-modes.md documentation

### v1.0.0 (2025-11-09)
- Initial release
- Core principles and workflows
- Project type detection for Next.js, Express, React Native, Flutter, FastAPI
- Advanced guides: rollback, secrets, migrations, dependencies, validation
- Adaptive autonomy system
- Quality standards and security checklists

---

## 🤝 Contributing

This is an internal instruction system. To propose changes:

1. Test changes in actual development scenarios
2. Ensure backward compatibility
3. Document rationale for changes
4. Update version number and changelog
5. Update all projects using the instructions

---

## 📊 Repository Structure

```
Claude_Instructions/                 (This repository)
├── README.md                        (This file)
└── claude_instructions/             (Copy this folder to projects)
    ├── claude_instructions.md
    ├── initialization.md
    ├── meta-modes.md
    ├── core/
    ├── workflows/
    ├── standards/
    ├── project-types/
    ├── advanced/
    └── .claude/
```

---

## ✅ Quick Reference

**Installation:**
```bash
cp -r claude_instructions/ /path/to/project/
```

**Initialization:**
```
INITIATE CLAUDE CODE INSTRUCTIONS
```

**Quick Start:**
```
QUICK START
```

**Switch Mode:**
```
SWITCH MODE: SPEED MODE
```

**Check Status:**
```
SHOW SESSION STATUS
```

**Reconfigure:**
```
RECONFIGURE SESSION
```

---

**Version:** 1.3.0
**License:** Internal Use
**Maintained By:** Development Team
