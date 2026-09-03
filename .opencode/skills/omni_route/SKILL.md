---
name: omni_route
description: Routes user requests to the optimal agent based on task classification (coding, reasoning, fast). Handles token-aware fallback, model fallback, and subagent chaining using ONLY NVIDIA models.
---

# Omni Router Skill

Analyzes user requests, classifies the task type, selects the appropriate agent, and delegates execution. Supports token-aware context management, model fallback chains, and subagent chaining.

## Allowed Models

| Model ID | Provider | Max Tokens |
|----------|----------|------------|
| nemotron-lightning | nvidia | 256,000 |
| nemotron-super | nvidia | 1,000,000 |
| mistral-nemotron | nvidia | 128,000 |

## Forbidden Models

Never route to:
- deepseek-v4-pro-0813
- OCR / ASR / safety / embedding / guardrail models

## Task Classification

| Category | Keywords / Indicators | Target Agent |
|----------|----------------------|--------------|
| **coding** | code, debug, fix, refactor, function, class, API, script, unit test, implement, build, deploy | coding-agent |
| **reasoning** | design, architecture, plan, analyze, workflow, strategy, multi-step, roadmap, "how should I", "plan for" | architect-agent |
| **fast** | summarize, quick, short, simple, explain, define, "what is", "brief", "tl;dr", simple question | fast-agent |

## Routing Logic

1. **Analyze** the user request for intent and complexity
2. **Classify** into one primary category (coding / reasoning / fast)
3. **Measure context length** (estimate tokens from conversation history + request)
4. **Select model** based on routing rules below
5. **Select agent** based on classification
6. **Check context** — if estimated tokens exceed model limit, compress/summarize and retry
7. **Delegate** to the selected agent with full context
8. **Chain** — if agent response indicates need for another specialty, route again (subagent chaining, max 3 hops)
9. **Log** routing decision with brief rationale

## Token Limits

- nemotron-super → 1,000,000 tokens
- nemotron-lightning → 256,000 tokens
- mistral-nemotron → 128,000 tokens

## Routing Rules

### 1. Coding Tasks
Keywords: code, debug, fix, refactor, function, class, API, script, unit test

- If context < 256k tokens → nemotron-lightning
- If context >= 256k tokens → nemotron-super

Fallbacks:
- mistral-nemotron
- nemotron-super

### 2. Reasoning / Architecture / Planning
Keywords: design, architecture, plan, analyze, workflow, strategy, multi-step

- If context < 1M tokens → nemotron-super
- If context >= 1M tokens → summarize + compress + retry with nemotron-super

Fallbacks:
- nemotron-lightning
- mistral-nemotron

### 3. Fast / Lightweight Tasks
Keywords: summarize, quick, short, simple, explain

- Primary → mistral-nemotron

Fallback:
- nemotron-lightning

### 4. Cost-Aware Routing (applies when context is small)
- If request < 500 chars → mistral-nemotron
- If request 500–3000 chars → nemotron-lightning
- If request > 3000 chars → nemotron-super

### 5. Token-Aware Fallback
If model returns token-limit error:
- Switch to next fallback model
- If context too large → compress + retry

### 6. Model Fallback Chain (per agent)

| Agent | Primary | Fallback 1 | Fallback 2 |
|-------|---------|------------|------------|
| coding-agent | nemotron-lightning | mistral-nemotron | nemotron-super |
| architect-agent | nemotron-super | nemotron-lightning | mistral-nemotron |
| fast-agent | mistral-nemotron | nemotron-lightning | — |

### 7. Fallback Trigger Conditions

Retry once, then switch to next model on:
- **Provider overload** (HTTP 503, "provider overloaded", "capacity exceeded")
- **Rate limit** (HTTP 429, "rate limit", "too many requests")
- **Timeout** (request timeout, chunk timeout)
- **Network failure** (connection refused, DNS error, socket hang up)
- **Model error** (model not found, invalid request, context length exceeded)

### 8. Fallback Retry Logic

1. **Attempt 1**: Primary model
2. **On failure**: Wait 2s, retry same model once
3. **On retry failure**: Switch to Fallback 1, log switch
4. **On Fallback 1 failure**: Wait 2s, retry Fallback 1 once
5. **On retry failure**: Switch to Fallback 2, log switch
6. **On Fallback 2 failure**: Return structured error

## Shared Fallback Handler

```
handle_model_failure(model_name, error, agent_name, attempt=1):
  1. Log: "Model {model_name} failed for {agent_name}: {error}"
  2. If attempt < 2:
       - Wait 2 seconds
       - Retry with same model
       - If success: return output
  3. Get fallback chain for agent_name
  4. Find next model in chain after model_name
  5. If next model exists:
       - Log: "Switching {agent_name} from {model_name} to {next_model}"
       - Invoke agent with next_model
       - Return handle_model_failure(next_model, error, agent_name, 1)
  6. Else:
       - Return structured error:
         {
           "error": "All models failed",
           "agent": agent_name,
           "attempted": [model_name, ...fallback_chain],
           "last_error": error,
           "suggestion": "Try again later or check provider status"
         }
```

## Subagent Chaining

- After agent response, check for explicit handoff signals (e.g., "NEEDS_CODER:", "NEEDS_ARCHITECT:")
- Parse and route to indicated agent
- Maximum 3 chained hops per request to prevent loops

## Output Format

```
🔀 Routing: [category] → [agent_name]
Reason: [1-line explanation]
Context tokens: [estimate]
Model: [primary_model] (fallback: [fallback_model] if used)
---
[agent output]
```

If chained:
```
🔗 Chain: [from_agent] → [to_agent]
Reason: [handoff reason]
Model: [model_used]
---
[agent output]
```

If fallback triggered:
```
⚠️ Fallback: [agent_name] switched from [primary] to [fallback]
Reason: [error_type]
---
[agent output]
```

## Context Compression

When context exceeds model limit:
1. Invoke fast-agent with "compress context preserving key decisions, code snippets, and facts"
2. Replace original context with compressed version
3. Retry with same model

## Logging

Every routing decision must log:
- Category classified
- Agent selected
- Model selected
- Context token estimate
- Request character count
- Any fallback/compression triggered