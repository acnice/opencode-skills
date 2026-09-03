---
name: grilling
description: >
  Stress-test a plan, decision, or idea using adversarial questioning.
  Build a dependency-ordered design tree, prioritize questions by importance,
  merge similar decisions, and auto-select recommended answers when appropriate.

capabilities:
  - reasoning
  - planning
  - critique
  - adversarial_questioning
  - design_tree_construction
  - decision_prioritization
  - similarity_detection
---

triggers:
  - "grill"
  - "stress test"
  - "challenge my plan"
  - "poke holes"
  - "interrogate"
  - "make this robust"

instructions:
  # --- CORE BEHAVIOR ---
  - >
    Construct a **design tree** from the user's plan:
      - Nodes = decisions or required facts
      - Edges = dependencies
      - Merge nodes that represent semantically similar decisions
      - Assign each node an **importance score**:
          * High: existential decisions, constraints, risks, irreversible choices
          * Medium: structural decisions, sequencing, resource allocation
          * Low: stylistic preferences, optional enhancements

  - >
    The **frontier** is every decision whose prerequisites are resolved.
    Sort the frontier by:
      1. Importance score (descending)
      2. Dependency depth (ascending)
      3. Novelty (ask new categories before refinements)

  - >
    Work in **rounds**:
      1. Recompute the frontier.
      2. Sort by importance.
      3. Ask every frontier question in order.
      4. Provide recommended answers.
      5. Wait for the user's answers before continuing.

  # --- RECOMMENDED ANSWER LOGIC ---
  - >
    When multiple frontier questions are similar or share a merged node:
      - Auto-select a single recommended answer for all of them.
      - Use the most conservative, risk-aware option.
      - Do not repeat the same question; ask it once.

  - >
    Recommended answers must be:
      - Realistic
      - Evidence-based
      - Aligned with best practices
      - Not idealistic or overly optimistic

  # --- FORMAT ---
  - >
    Format each round EXACTLY like this:

    ❓ **Q1 — <question title>**  
    <question body, may be multiple paragraphs, may include multiple choices>  
    ➡️ <your recommended answer>

    ---
    ❓ **Q2 — <question title>**  
    <question body>  
    ➡️ <your recommended answer>

  # --- FACT-FINDING RULE ---
  - >
    Finding facts is YOUR job, never the user's.
    If a frontier question requires external facts (filesystem, tools, APIs),
    dispatch a sub-agent to gather them.
    A running fact-finding task counts as an unresolved prerequisite.