# opencode-skills

A collection of reusable skills for [opencode](https://opencode.ai) — specialized instructions that extend opencode's capabilities for specific tasks.

## Available Skills

| Skill | Description | Use Case |
|-------|-------------|----------|
| **business-idea-planner** | Transforms business ideas into revenue-optimised, AI-enhanced, legally-aware implementation plans tailored for Australia | Refining business concepts, planning SaaS/platform launches, validating ideas for the Australian market |
| **grilling** | Stress-test a plan, decision, or idea using adversarial questioning. Build a dependency-ordered design tree, prioritize questions by importance, merge similar decisions, and auto-select recommended answers | "grill my plan", "stress test this", "challenge my idea", "poke holes in this" |
| **code-review** | Two-axis review (Standards + Spec) of all changes since a fixed point. Runs parallel sub-agents for React/TS/CDK/AWS/GitHub Actions standards and spec compliance | "review since main", "code review this PR", "audit this diff" |
| **make-plan** | Create comprehensive, implementation-ready plans with TDD-driven tasks, exact file paths, commands, and expected outputs. Designed for engineers with minimal context | "make a plan", "write a plan", "implementation plan", "how do I build this" |
| **implement-feature** | Implement features using community-best patterns: TypeScript strict, React hooks, CDK constructs, DynamoDB, GitHub Actions, Jest/RTL, ESLint/Prettier, pre-commit hooks. Mandatory review-before-commit | "implement this feature", "build this", "add endpoint", "cdk stack change" |

---

## Right model for the right task: model-pinned subagents

Routing is **not** a skill. A skill is prose injected into the model that is already running, so it cannot measure a token budget, re-route its own turn, or react to HTTP 429/503 errors — those are runtime concerns. opencode already ships the correct mechanism: a strong primary agent that delegates narrow, token-heavy, low-reasoning work to cheaper **subagents** via the built-in `task` tool. Each subagent pins its own model in config.

The seam is the `task` tool boundary: the primary decides *what* to delegate, and the subagent's own config decides *which* model runs it. The primary cannot pass a model per call — the model is fixed per subagent config, so tiering is expressed as distinct subagents, not as one subagent the parent re-points.

### Tiers

| Subagent | Model | Purpose |
|----------|-------|---------|
| `explore` | `deepseek/deepseek-v4-flash` | Broad, read-only codebase reconnaissance and summaries |
| `implement` | `deepseek/deepseek-v4-flash` | Scoped code changes and tests |
| `deep` | `deepseek/deepseek-v4-pro` | Hard, multi-step reasoning and architecture analysis |

**Cost mechanism:** a plan/explore turn that would otherwise burn expensive primary-model tokens reading 20 files spends cheap tokens in a child session and returns a short summary to the parent. The parent's context stays small, and the expensive per-token rate applies only to genuine reasoning.

**Avoid over-delegation:** the round-trip overhead of spinning a child session and summarising can cost more than a direct read. Delegate multi-file or open-ended discovery; read a single known file directly.

### Verifying the model tiers

Restart opencode after changing config (it loads once at startup), then run a delegation smoke test:

> Smoke-test the subagent model tiers. Spawn @explore to locate and summarize where auth/API-key handling lives, spawn @implement to append a single innocuous log line to one file then revert it, and spawn @deep to reason about the tradeoff between retry-with-backoff vs circuit-breaker. Report back what each subagent returned.

Then check the `mode=subagent` stream lines in `~/.local/share/opencode/log/opencode.log` and confirm each agent streamed its pinned model:

- `explore` → `providerID=deepseek modelID=deepseek-v4-flash`
- `implement` → `providerID=deepseek modelID=deepseek-v4-flash`
- `deep` → `providerID=deepseek modelID=deepseek-v4-pro`

If an agent streams a different model, the config wasn't picked up (restart) or the model reference is malformed.

### Global config example (`~/.config/opencode/opencode.json`)

Configure at global scope so it applies everywhere. opencode's real schema uses `agent` (singular), `provider` (singular), `small_model`, and `default_agent`. Set `small_model` so title/summary/compaction system agents use a cheap model instead of the driver.

```json
{
  "$schema": "https://opencode.ai/config.json",
  "small_model": "nvidia/nvidia/nemotron-3.5-lightning-30b-a3b",
  "provider": {
    "nvidia": {
      "options": {
        "baseURL": "https://integrate.api.nvidia.com/v1",
        "apiKey": "{env:NVIDIA_API_KEY}"
      }
    },
    "deepseek": {
      "options": {
        "baseURL": "https://api.deepseek.com",
        "apiKey": "{env:DEEPSEEK_API_KEY}"
      }
    }
  },
  "agent": {
    "explore": {
      "description": "Broad read-only codebase reconnaissance: multi-file search, tracing where things live, reading docs/logs, and returning a short cited summary.",
      "mode": "subagent",
      "model": "deepseek/deepseek-v4-flash",
      "permission": { "edit": "deny", "bash": "deny", "task": "deny" }
    },
    "implement": {
      "description": "Focused code changes: implement, refactor, fix, and add tests once the change is understood and scoped.",
      "mode": "subagent",
      "model": "deepseek/deepseek-v4-flash",
      "permission": { "edit": "allow", "bash": "allow", "task": "deny" }
    },
    "deep": {
      "description": "Deep, multi-step reasoning for hard problems: architecture and trade-off analysis, gnarly debugging, and planning.",
      "mode": "subagent",
      "model": "deepseek/deepseek-v4-pro",
      "permission": { "edit": "deny", "bash": "allow", "task": "deny" }
    }
  }
}
```

Notes:

- Model IDs always carry a provider prefix, e.g. `deepseek/deepseek-v4-flash`.
- Use `{env:VAR}` interpolation for secrets; never hardcode API keys in config.
- After saving config, **quit and restart opencode** — config is loaded once at startup.
- Verify model IDs with `opencode models` and the merged config with `opencode debug config`.
- Automatic model fallback on provider errors is **not** exposed to config today; do not reimplement it in prose.

### Skills that use the seam

`make-plan` delegates codebase discovery to `@explore` before drafting a plan, so discovery cost lands on the cheap model.

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
    "paths": ["./node_modules/@your-org/opencode-skill-business-idea-planner"]
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
