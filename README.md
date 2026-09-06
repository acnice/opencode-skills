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

## omni_route Skill — Deep Dive

The `omni_route` skill implements an intelligent request router that classifies incoming tasks and delegates to specialized agents. This enables optimal model selection per task type and provides resilience through model fallback chains.

### Architecture Overview

```
User Request
    │
    ▼
┌─────────────────────────────────────┐
│         Omni Router Agent           │  (omni_router)
│  • Classifies task type             │
│  • Estimates context tokens         │
│  • Selects target agent             │
│  • Handles fallback chains          │
└─────────────────────────────────────┘
    │
    ├── planning ──────► strategist_agent
    ├── coding ────────► coder_agent
    ├── lightweight ──► lightweight_agent
    ├── multimodal ───► vision_agent
    └── retrieval ────► retrieval_agent
```

### Task Classification Rules

| Category | Keywords / Indicators | Target Agent |
|----------|----------------------|--------------|
| **planning** | business plan, strategy, legal, compliance, feature ideation, roadmap, architecture decisions, "how should I", "plan for" | strategist_agent |
| **coding** | code, debug, refactor, implement, API, database, architecture, test, build, deploy, "write a function", "fix this error" | coder_agent |
| **lightweight** | summarize, quick answer, explain, define, "what is", "brief", "tl;dr", simple question | lightweight_agent |
| **multimodal** | image, diagram, mockup, UI design, screenshot, visual, chart, graph, "draw", "design a screen" | vision_agent |
| **retrieval** | search, find, lookup, RAG, embeddings, semantic search, "find documents", "retrieve", "similar to" | retrieval_agent |

### Model Fallback Chains

Each agent has a primary model with automatic fallback on failure:

| Agent | Primary | Fallback 1 | Fallback 2 |
|-------|---------|------------|------------|
| **strategist_agent** | nemotron-3-ultra-550b | nemotron-3.5-lightning-30b | deepseek-flash (paid) |
| **coder_agent** | nemotron-3.5-lightning-30b | nemotron-3-ultra-550b | deepseek-flash (paid) |
| **lightweight_agent** | nemotron-3.5-lightning-30b | nemotron-3-ultra-550b | deepseek-flash (paid) |
| **vision_agent** | nemotron-3-ultra-550b | nemotron-3.5-lightning-30b | deepseek-flash (paid) |
| **retrieval_agent** | nemotron-3.5-lightning-30b | nemotron-3-ultra-550b | deepseek-flash (paid) |

**Fallback Triggers:** Provider overload (503), rate limit (429), timeout, network failure, model error (context exceeded, invalid request).

### Token-Aware Context Management

- **Threshold:** 900k tokens
- **Action:** If context > threshold, invoke `lightweight_agent` to summarize, then re-route with condensed context
- **Preserved:** Key facts, decisions, code snippets, file paths

### Subagent Chaining

- **Max hops:** 3 per request
- **Trigger:** Explicit handoff signals in agent output (e.g., `NEEDS_CODER:`, `NEEDS_STRATEGIST:`)
- **Loop prevention:** Tracks hop count

---

## Configuring Models for Specific Work

The routing stack uses a fixed pool of five models (see `opencode.config.json` in this repo):

| Alias | Provider | Tier | Model ID |
|-------|----------|------|----------|
| nemotron-ultra | nvidia | free | nvidia/nemotron-3-ultra-550b-a55b |
| nemotron-lightning | nvidia | free | nvidia/nemotron-3.5-lightning-30b-a3b |
| deepseek-flash | deepseek | paid | deepseek-v4-flash |
| deepseek-pro | deepseek | paid | deepseek-v4-pro |
| deepseek-vision | deepseek | paid | deepseek-v4-flash-vision-exp |

Free tier (Nemotron lightning/ultra) is the default for most work. DeepSeek (paid) is only used for vision or when context exceeds the 256k Nemotron window and the 1M DeepSeek window is required.

### 1. Global Model Configuration (opencode.json)

Define your models once, reference by alias:

```json
{
  "providers": {
    "nvidia": {
      "baseURL": "https://integrate.api.nvidia.com/v1",
      "apiKey": "${NVIDIA_API_KEY}",
      "mode": "raw"
    },
    "deepseek": {
      "baseURL": "https://api.deepseek.com",
      "apiKey": "${DEEPSEEK_API_KEY}",
      "mode": "raw"
    }
  },
  "models": {
    "nemotron-ultra": { "provider": "nvidia", "model": "nvidia/nemotron-3-ultra-550b-a55b" },
    "nemotron-lightning": { "provider": "nvidia", "model": "nvidia/nemotron-3.5-lightning-30b-a3b" },
    "deepseek-flash": { "provider": "deepseek", "model": "deepseek-v4-flash" },
    "deepseek-pro": { "provider": "deepseek", "model": "deepseek-v4-pro" },
    "deepseek-vision": { "provider": "deepseek", "model": "deepseek-v4-flash-vision-exp" }
  }
}
```

### 2. Per-Agent Model Assignment

Assign the best model for each agent's workload:

```json
{
  "agents": {
    "strategist_agent": {
      "type": "subagent",
      "model": "nemotron-ultra",        // Complex reasoning, large context
      "system": "..."
    },
    "coder_agent": {
      "type": "subagent",
      "model": "nemotron-lightning",    // Fast, code-focused
      "system": "..."
    },
    "lightweight_agent": {
      "type": "subagent",
      "model": "nemotron-lightning",    // Cheap, fast for simple Q&A
      "system": "..."
    },
    "vision_agent": {
      "type": "subagent",
      "model": "deepseek-vision",       // Vision-capable (paid)
      "system": "..."
    },
    "retrieval_agent": {
      "type": "subagent",
      "model": "nemotron-lightning",    // Fast search/synthesis
      "system": "..."
    },
    "omni_router": {
      "type": "subagent",
      "model": "nemotron-lightning",    // Fast classification
      "system": "..."
    }
  }
}
```

### 3. Recommended Model-to-Agent Mapping

| Agent | Workload Characteristics | Recommended Model Traits | Example Models |
|-------|-------------------------|--------------------------|----------------|
| **strategist_agent** | Deep analysis, multi-step planning, legal/compliance, large context | High reasoning, large context window | nemotron-ultra, deepseek-pro |
| **coder_agent** | Code generation, debugging, refactoring, API design | Fast inference, code-specialized, strict output | nemotron-lightning, deepseek-flash |
| **lightweight_agent** | Summaries, definitions, quick answers | Low latency, cheap, concise | nemotron-lightning |
| **vision_agent** | Diagram design, UI mockups, visual reasoning | Vision-capable | deepseek-vision, nemotron-ultra |
| **retrieval_agent** | Code search, pattern matching, synthesis | Fast, good at synthesis | nemotron-lightning, deepseek-flash |
| **omni_router** | Classification, token estimation, routing logic | Fast, reliable classification | nemotron-lightning, deepseek-flash |

### 4. Environment-Specific Overrides

Use different models for dev vs prod (both within the allowed pool):

```json
// .opencode/opencode.json (committed)
{
  "models": {
    "coder-model": { "provider": "nvidia", "model": "nvidia/nemotron-3.5-lightning-30b-a3b" }
  },
  "agents": {
    "coder_agent": { "model": "coder-model" }
  }
}

// .opencode/opencode.local.json (gitignored - per-developer overrides)
{
  "models": {
    "coder-model": { "provider": "deepseek", "model": "deepseek-v4-flash" }
  }
}
```

### 5. Cost Optimization Strategies

| Strategy | Implementation |
|----------|----------------|
| **Free-first default** | Route all small/medium work to Nemotron lightning/ultra (free) |
| **Paid only when needed** | Use DeepSeek for vision, >256k context, or after free fallbacks are exhausted |
| **Local dev overrides** | Use `.opencode.local.json` to point specific agents at DeepSeek only in dev |
| **Request routing** | Route simple Q&A to lightweight_agent automatically |
| **Context summarization** | Auto-summarize at 900k tokens to stay within free model windows |

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