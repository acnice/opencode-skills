# opencode-skills

A collection of reusable skills for [opencode](https://opencode.ai) — specialized instructions that extend opencode's capabilities for specific tasks.

## Available Skills

| Skill | Description | Use Case |
|-------|-------------|----------|
| **business-idea-planner** | Transforms business ideas into revenue-optimised, AI-enhanced, legally-aware implementation plans tailored for Australia | Refining business concepts, planning SaaS/platform launches, validating ideas for the Australian market |
| **omni_route** | Routes user requests to the optimal agent based on task classification (planning, coding, lightweight, multimodal, retrieval) | All incoming requests to delegate to specialized agents with token-aware fallback and model fallback chains |
| **grilling** | Stress-test a plan, decision, or idea using adversarial questioning. Build a dependency-ordered design tree, prioritize questions by importance, merge similar decisions, and auto-select recommended answers | "grill my plan", "stress test this", "challenge my idea", "poke holes in this" |
| **code-review** | Two-axis review (Standards + Spec) of all changes since a fixed point. Runs parallel sub-agents for React/TS/CDK/AWS/GitHub Actions standards and spec compliance | "review since main", "code review this PR", "audit this diff" |
| **writing-plans** | Create comprehensive, implementation-ready plans with TDD-driven tasks, exact file paths, commands, and expected outputs. Designed for engineers with minimal context | "make a plan", "write a plan", "implementation plan", "how do I build this" |
| **implement-feature** | Implement features using community-best patterns: TypeScript strict, React hooks, CDK constructs, DynamoDB, GitHub Actions, Jest/RTL, ESLint/Prettier, pre-commit hooks. Mandatory review-before-commit | "implement this feature", "build this", "add endpoint", "cdk stack change" |

---

## How to Add These Skills to Other Projects/Repos

### Method 1: Copy the Skill Directory (Recommended)

Each skill is self-contained in its own directory under `.opencode/skills/`. To use a skill in another project:

```bash
# From your target project root
mkdir -p .opencode/skills

# Copy the skill directory
cp -r /path/to/opencode-skills/.opencode/skills/business-idea-planner .opencode/skills/
# Or copy all skills at once
cp -r /path/to/opencode-skills/.opencode/skills/* .opencode/skills/
```

### Method 2: Git Submodule (For Syncing Updates)

If you want to keep skills in sync with this repository:

```bash
# From your target project root
git submodule add https://github.com/your-org/opencode-skills.git .opencode/skills/opencode-skills

# Create symlinks to the skills you want to use
mkdir -p .opencode/skills
ln -s ../skills/opencode-skills/.opencode/skills/business-idea-planner .opencode/skills/business-idea-planner
ln -s ../skills/opencode-skills/.opencode/skills/omni_route .opencode/skills/omni_route
ln -s ../skills/opencode-skills/.opencode/skills/grilling .opencode/skills/grilling
ln -s ../skills/opencode-skills/.opencode/skills/code-review .opencode/skills/code-review
ln -s ../skills/opencode-skills/.opencode/skills/make-plan .opencode/skills/make-plan
ln -s ../skills/opencode-skills/.opencode/skills/implement-feature .opencode/skills/implement-feature
```

### Method 3: npm/yarn/pnpm Package (For Distribution)

To publish as a package:

1. Create a `package.json` in each skill directory:
```json
{
  "name": "@your-org/opencode-skill-business-idea-planner",
  "version": "1.0.0",
  "files": ["SKILL.md"],
  "publishConfig": { "access": "public" }
}
```

2. Publish to npm:
```bash
cd .opencode/skills/business-idea-planner
npm publish
```

3. Install in target project:
```bash
npm install @your-org/opencode-skill-business-idea-planner
```

4. Configure opencode to use it (in `.opencode/opencode.json`):
```json
{
  "skills": {
    "business-idea-planner": "./node_modules/@your-org/opencode-skill-business-idea-planner/SKILL.md"
  }
}
```

---

## Skill Structure

Each skill follows this structure:

```
.opencode/skills/
└── skill-name/
    └── SKILL.md          # Main skill definition (required)
```

The `SKILL.md` file must start with frontmatter:
```markdown
---
name: skill-name
description: Brief description of what the skill does and when to use it
---
```

---

## Using Skills in opencode

Once added to your project's `.opencode/skills/` directory, skills are automatically available. Use them by:

1. **Explicit invocation**: Reference the skill by name in your prompt
2. **Auto-selection**: opencode will automatically select relevant skills based on context

---

## Creating Your Own Skills

1. Create a new directory under `.opencode/skills/your-skill-name/`
2. Add a `SKILL.md` file with frontmatter and instructions
3. Follow the patterns in existing skills:
   - Clear description of behavior
   - Step-by-step instructions
   - Output format specifications
   - Examples where helpful

---

## Contributing

Feel free to submit PRs with new skills or improvements to existing ones!