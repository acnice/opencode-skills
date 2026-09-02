---
name: omni_route
description: Routes user requests to the optimal agent based on task classification (planning, coding, lightweight, multimodal, retrieval). Handles token-aware fallback, model fallback, and subagent chaining. Use for all incoming requests to delegate to specialized agents.
---

# Omni Router Skill

Analyzes user requests, classifies the task type, selects the appropriate agent, and delegates execution. Supports token-aware context management, model fallback chains, and subagent chaining.

## Task Classification

| Category | Keywords / Indicators | Target Agent |
|----------|----------------------|--------------|
| **planning** | business plan, strategy, legal, compliance, feature ideation, roadmap, architecture decisions, "how should I", "plan for" | strategist_agent |
| **coding** | code, debug, refactor, implement, API, database, architecture, test, build, deploy, "write a function", "fix this error" | coder_agent |
| **lightweight** | summarize, quick answer, explain, define, "what is", "brief", "tl;dr", simple question | lightweight_agent |
| **multimodal** | image, diagram, mockup, UI design, screenshot, visual, chart, graph, "draw", "design a screen" | vision_agent |
| **retrieval** | search, find, lookup, RAG, embeddings, semantic search, "find documents", "retrieve", "similar to" | retrieval_agent |

## Routing Logic

1. **Analyze** the user request for intent and complexity
2. **Classify** into one primary category (planning/coding/lightweight/multimodal/retrieval)
3. **Select** the corresponding agent
4. **Check context** — if estimated tokens > 900k, summarize context and route to lightweight_agent for condensation before delegating
5. **Delegate** to the selected agent with full context
6. **Chain** — if agent response indicates need for another specialty, route again (subagent chaining)
7. **Log** routing decision with brief rationale

## Token-Aware Fallback

- Estimate context tokens before delegation
- If > 900k tokens: invoke lightweight_agent to summarize, then re-route with condensed context
- Preserve key facts, decisions, and code snippets in summary

## Model Fallback Chain

### Fallback Sequence per Agent

| Agent | Primary | Fallback 1 | Fallback 2 |
|-------|---------|------------|------------|
| coder_agent | deepseek/deepseek-v4-flash-0731 | nvidia/nemotron-3.5-lightning-30b-a3b | deepseek/deepseek-v4-pro-0813 |
| strategist_agent | nvidia/nemotron-3.5-lightning-30b-a3b | deepseek/deepseek-v4-flash-0731 | deepseek/deepseek-v4-pro-0813 |
| lightweight_agent | nvidia/llama-3.1-nemotron-safety-guard-8b-v3 | nvidia/nemotron-3.5-lightning-30b-a3b | deepseek/deepseek-v4-flash-0731 |
| vision_agent | nvidia/muse-glimmer-30b | nvidia/nemotron-3.5-lightning-30b-a3b | deepseek/deepseek-v4-flash-0731 |
| retrieval_agent | nvidia/nemotron-3-embed-1b | nvidia/nemotron-3.5-lightning-30b-a3b | deepseek/deepseek-v4-flash-0731 |

### Fallback Trigger Conditions

Retry once, then switch to next model on:
- **Provider overload** (HTTP 503, "provider overloaded", "capacity exceeded")
- **Rate limit** (HTTP 429, "rate limit", "too many requests")
- **Timeout** (request timeout, chunk timeout)
- **Network failure** (connection refused, DNS error, socket hang up)
- **Model error** (model not found, invalid request, context length exceeded)

### Fallback Retry Logic

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

- After agent response, check for explicit handoff signals (e.g., "NEEDS_CODER:", "NEEDS_STRATEGIST:")
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