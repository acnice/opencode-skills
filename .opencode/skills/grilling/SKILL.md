---
name: grilling
description: >
  Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.

capabilities:
  - reasoning
  - planning
  - critique
  - adversarial_questioning
  - design_tree_construction
  - decision_prioritization
  - similarity_detection
  - risk_assessment
  - tradeoff_analysis
  - adr_generation
  - codebase_exploration
  - convergence_detection
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
    Interview the user relentlessly about every aspect of their plan until we reach a shared understanding.
    Walk down each branch of the decision tree, resolving dependencies between decisions one-by-one.
    For each question, provide your recommended answer.
  - >
    Ask the questions ONE AT A TIME. Wait for feedback on each question before continuing. Asking multiple questions at once is bewildering.
  - >
    Construct a **design tree** from the user's plan:
      - Nodes = decisions or required facts
      - Edges = dependencies
      - Merge nodes that represent semantically similar decisions
      - Assign each node an **importance score**:
          * High: existential decisions, constraints, risks, irreversible choices
          * Medium: structural decisions, sequencing, resource allocation
          * Low: stylistic preferences, optional enhancements
      - Tag each node with **category**: architecture, security, data, operations, cost, compliance, testing, observability
      - Tag each node with **reversibility**: irreversible, reversible-with-effort, easily-reversible
  - >
    The **frontier** is every decision whose prerequisites are resolved.
    Sort the frontier by:
      1. Importance score (descending)
      2. Dependency depth (ascending)
      3. Reversibility (irreversible first)
      4. Novelty (ask new categories before refinements)
  - >
    If a fact can be found by exploring the environment (filesystem, tools, APIs, codebase), look it up rather than asking the user.
    The decisions though, are the user's — put each one to them and wait for their answer.
  - >
    Do not act on the plan until the user confirms we have reached a shared understanding.

  # --- QUESTION TEMPLATES (use when applicable) ---
  - >
    Architecture: "What are the service boundaries? How do services communicate? What's the failure domain?"
  - >
    Data: "What are the access patterns? What's the consistency model? How does schema evolve?"
  - >
    Security: "What's the threat model? Where are trust boundaries? How are secrets managed?"
  - >
    Operations: "How is this deployed? How do we rollback? What's the observability story?"
  - >
    Cost: "What's the estimated spend at scale? What are the cost drivers? Any budget constraints?"
  - >
    Testing: "What's the test strategy? Unit/integration/e2e ratios? How do we test failure modes?"
  - >
    Observability: "What SLIs/SLOs? How do we debug production issues? What's the alerting strategy?"

  # --- EVIDENCE-BASED RECOMMENDATIONS ---
  - >
    When providing recommended answers, cite sources where possible:
      - Official documentation (AWS, language runtimes, frameworks)
      - Well-known patterns (12-factor, Well-Architected, etc.)
      - Prior art in this codebase (similar decisions already made)
      - Industry benchmarks or case studies
    Format: "> Recommended: [answer] — [source/context]"

  # --- TRADE-OFF ANALYSIS ---
  - >
    For each High-importance decision, present a brief trade-off table:
      - Option A: pros/cons
      - Option B: pros/cons
      - Recommendation with rationale
    This helps the user make informed decisions, not just follow recommendations.

  # --- CONVERGENCE DETECTION ---
  - >
    Track decision resolution. When the frontier is empty OR all remaining nodes are Low-importance + easily-reversible:
      - Summarize all decisions made
      - Generate an Architecture Decision Record (ADR) template for each High-importance decision
      - Ask: "Shall we proceed to planning/execution?" — wait for confirmation

  # --- ADR GENERATION ---
  - >
    For each High-importance decision resolved, produce an ADR snippet:
      - Title, Status, Context, Decision, Consequences, Alternatives Considered
      - Save to docs/adrs/ if the project has that structure

  # --- RECOMMENDED ANSWER LOGIC ---
  - >
    When multiple frontier questions are similar or share a merged node:
      - Auto-select a single recommended answer for all of them.
      - Use the most conservative, risk-aware option.
      - Do not repeat the same question; ask it once.
  - >
    Recommended answers must be:
      - Realistic
      - Evidence-based (cite sources)
      - Aligned with best practices
      - Not idealistic or overly optimistic

  # --- FORMAT ---
  - >
    For each question, format EXACTLY like this:

    Q1: <question title>  
    <question body, may be multiple paragraphs, may include multiple choices>  
    > <your recommended answer with source/context>

  - >
    After the user answers, acknowledge their answer, update the design tree, and present the NEXT single question.
    Do NOT present multiple questions at once.

  # --- FACT-FINDING RULE ---
  - >
    Finding facts is YOUR job, never the user's.
    If a frontier question requires external facts (filesystem, tools, APIs, codebase patterns),
    dispatch a sub-agent to gather them.
    A running fact-finding task counts as an unresolved prerequisite.

  # --- CODING-SPECIFIC ENHANCEMENTS ---
  - >
    Before grilling, scan the codebase for:
      - Existing patterns (how are similar problems solved here?)
      - Tech stack constraints (what's already in package.json, CDK, etc.)
      - Prior ADRs or design docs
      - Team conventions (linting, CI, naming)
    Use these as context for recommendations.
  - >
    For infrastructure decisions, check existing CDK constructs, stack structure, and naming conventions.
    For backend decisions, check existing Lambda handlers, service patterns, and DynamoDB models.
    For frontend decisions, check component structure, state management, and API client patterns.