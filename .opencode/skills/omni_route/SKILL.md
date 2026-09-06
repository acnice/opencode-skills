---
name: omni_route
description: Routes user requests to the optimal agent based on task classification (coding, reasoning, fast, multimodal, retrieval). Handles token-aware fallback, model fallback, and subagent chaining using NVIDIA (free) and DeepSeek (paid) models.
---

# Omni Router Skill

Analyzes user requests, classifies the task type, selects the appropriate agent, and delegates execution. Supports token-aware context management, model fallback chains, and subagent chaining.

## Allowed Models

Free tier = NVIDIA Nemotron. Paid tier = DeepSeek. Prefer free unless context or capabilities require DeepSeek.

| Model ID | Provider | Tier | Max Tokens |
|----------|----------|------|------------|
| nemotron-lightning | nvidia | free | 256,000 |
| nemotron-ultra | nvidia | free | 256,000 |
| deepseek-flash | deepseek | paid | 1,000,000 |
| deepseek-pro | deepseek | paid | 1,000,000 |
| deepseek-vision | deepseek | paid | 1,000,000 |

## Forbidden Models

Never route to:
- OCR / ASR / safety / embedding / guardrail models
- Deprecated/unavailable models (e.g. mistral-nemotron, nemotron-super)

## Task Classification

| Category | Keywords / Indicators | Target Agent |
|----------|----------------------|--------------|
| **coding** | code, debug, fix, refactor, function, class, API, script, unit test, implement, build, deploy | coder_agent |
| **reasoning** | design, architecture, plan, analyze, workflow, strategy, multi-step, roadmap, "how should I", "plan for" | strategist_agent |
| **fast** | summarize, quick, short, simple, explain, define, "what is", "brief", "tl;dr", simple question | lightweight_agent |
| **multimodal** | image, screenshot, diagram, mockup, UI, visual, chart, graph, vision | vision_agent |
| **retrieval** | find, search, locate, grep, glob, "where is", "how does", trace, dependency | retrieval_agent |

## Routing Logic

1. **Analyze** the user request for intent and complexity
2. **Classify** into one primary category (coding / reasoning / fast / multimodal / retrieval)
3. **Measure context length** (estimate tokens from conversation history + request)
4. **Select model** based on routing rules below
5. **Select agent** based on classification
6. **Check context** — if estimated tokens exceed model limit, compress/summarize and retry
7. **Delegate** to the selected agent with full context
8. **Chain** — if agent response indicates need for another specialty, route again (subagent chaining, max 3 hops)
9. **Log** routing decision with brief rationale

## Token Limits

- nemotron-ultra → 256,000 tokens
- nemotron-lightning → 256,000 tokens
- deepseek-flash → 1,000,000 tokens
- deepseek-pro → 1,000,000 tokens
- deepseek-vision → 1,000,000 tokens

## Routing Rules

### 1. Coding Tasks
Keywords: code, debug, fix, refactor, function, class, API, script, unit test, implement, build, deploy

- If context < 256k tokens → nemotron-lightning (free)
- If context >= 256k tokens → deepseek-flash (paid)

Fallbacks:
- nemotron-ultra
- deepseek-pro

### 2. Reasoning / Architecture / Planning
Keywords: design, architecture, plan, analyze, workflow, strategy, multi-step, roadmap

- If context < 256k tokens → nemotron-ultra (free)
- If context >= 256k tokens → deepseek-pro (paid)

Fallbacks:
- nemotron-lightning
- deepseek-flash

### 3. Fast / Lightweight Tasks
Keywords: summarize, quick, short, simple, explain, define, "what is", "brief", "tl;dr"

- Primary → nemotron-lightning (free)

Fallback:
- nemotron-ultra

### 4. Multimodal / Vision Tasks
Keywords: image, screenshot, diagram, mockup, UI, visual, chart, graph, vision

- Primary → deepseek-vision (paid)

Fallback:
- nemotron-ultra (text-only fallback)

### 5. Retrieval Tasks
Keywords: find, search, locate, grep, glob, "where is", "how does", trace, dependency

- If context < 256k tokens → nemotron-lightning (free)
- If context >= 256k tokens → deepseek-flash (paid)

Fallbacks:
- nemotron-ultra
- deepseek-pro

### 6. Cost-Aware Routing (free-first, applies when context is small)
- If request < 500 chars → nemotron-lightning
- If request 500–3000 chars → nemotron-ultra
- If request > 3000 chars → deepseek-flash

### 7. Token-Aware Fallback
If model returns token-limit error:
- Switch to next fallback model
- If context too large → compress + retry

### 8. Model Fallback Chain (per agent)

| Agent | Primary | Fallback 1 | Fallback 2 | Fallback 3 |
|-------|---------|------------|------------|------------|
| coder_agent | nemotron-lightning | nemotron-ultra | deepseek-flash | deepseek-pro |
| strategist_agent | nemotron-ultra | nemotron-lightning | deepseek-pro | deepseek-flash |
| lightweight_agent | nemotron-lightning | nemotron-ultra | — | — |
| vision_agent | deepseek-vision | nemotron-ultra | — | — |
| retrieval_agent | nemotron-lightning | nemotron-ultra | deepseek-flash | deepseek-pro |

## Fallback Trigger Conditions

Retry once, then switch to next model on:
- **Provider overload** (HTTP 503, "provider overloaded", "capacity exceeded")
- **Rate limit** (HTTP 429, "rate limit", "too many requests")
- **Timeout** (request timeout, chunk timeout)
- **Network failure** (connection refused, DNS error, socket hang up)
- **Model error** (model not found, invalid request, context length exceeded)

## Fallback Retry Logic

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

The omni_router agent coordinates chaining: after delegating, inspect the response for an explicit handoff signal and route again through the router.
- Example handoff signals: "NEEDS_CODER:", "NEEDS_STRATEGIST:", "NEEDS_LIGHTWEIGHT:", "NEEDS_VISION:", "NEEDS_RETRIEVAL:"
- Parse and route to the indicated agent (all names above are real subagents)
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
1. Invoke lightweight_agent with "compress context preserving key decisions, code snippets, and facts"
2. Replace original context with compressed version
3. Retry with same model

## Logging

Every routing decision must log:
- Category classified
- Agent selected
- Model selected (and tier: free/paid)
- Context token estimate
- Request character count
- Any fallback/compression triggered
