---
name: code-review
description: >
  Perform a two-axis review of all changes since a fixed point (commit, branch,
  tag, or merge-base). Axis 1: Standards — does the code follow repo conventions,
  React/TypeScript best practices, AWS/CDK patterns, and GitHub Actions security
  guidelines? Axis 2: Spec — does the code faithfully implement the originating
  issue/spec? Both axes run in parallel sub-agents and are aggregated cleanly.

capabilities:
  - code_analysis
  - diff_inspection
  - standards_enforcement
  - spec_validation
  - parallel_subagents
  - smell_detection
  - git_operations
  - cloud_architecture_review
  - frontend_review
---

triggers:
  - "review since"
  - "review this branch"
  - "review this PR"
  - "code review"
  - "audit this diff"

instructions:
  # --- FIXED POINT RESOLUTION ---
  - >
    Require a fixed point (commit SHA, branch, tag). Resolve via `git rev-parse`.
    Fail early if invalid. Capture diff via:
      git diff <fixed-point>...HEAD
    Capture commit list via:
      git log <fixed-point>..HEAD --oneline
    Stop if diff is empty.

  # --- SPEC SOURCE DISCOVERY ---
  - >
    Identify spec via:
      1. Issue references in commits (#123, Closes #45)
      2. User-supplied path
      3. docs/, specs/, .scratch/ matching branch/feature
      4. Ask user if none found
    If user says "no spec", Spec axis reports "No spec available".

  # --- STANDARDS SOURCE DISCOVERY ---
  - >
    Identify standards sources:
      - CODING_STANDARDS.md
      - CONTRIBUTING.md
      - React conventions (hooks rules, component purity, key usage)
      - TypeScript conventions (strict types, no implicit any, discriminated unions)
      - CDK best practices (construct boundaries, env separation, least privilege)
      - AWS Well-Architected (security, reliability, cost, performance)
      - GitHub Actions hardening (permissions: write-none default, OIDC, no secrets in env)
      - Tooling-enforced rules (skip these)

  - >
    Always include the **industry smell baseline**:
      - Mysterious Name
      - Duplicated Code
      - Feature Envy
      - Data Clumps
      - Primitive Obsession
      - Repeated Switches
      - Shotgun Surgery
      - Divergent Change
      - Speculative Generality
      - Message Chains
      - Middle Man
      - Refused Bequest

    Repo standards override the baseline.

  # --- PARALLEL SUB-AGENTS ---
  - >
    Spawn two sub-agents:

    **Standards Sub-Agent Prompt:**
      Include:
        - diff command + commit list
        - standards sources
        - smell baseline
      Brief:
        "Report per file/hunk:
         (a) violations of documented standards (cite file + rule)
         (b) React/TS/CDK/AWS/GitHub Actions best-practice violations
         (c) any baseline smell (name + quote hunk)
         Distinguish hard violations vs judgement calls.
         Skip tooling-enforced rules.
         Under 500 words."

    **Spec Sub-Agent Prompt:**
      Include:
        - diff command + commit list
        - spec contents or path
      Brief:
        "Report:
         (a) missing or partial requirements
         (b) behaviour not asked for (scope creep)
         (c) incorrect implementations of spec requirements
         Quote spec lines.
         Under 400 words."

    If spec missing: skip Spec axis.

  # --- AGGREGATION ---
  - >
    Aggregate results under:

    ## Standards
    <verbatim or lightly cleaned Standards report>

    ## Spec
    <verbatim or lightly cleaned Spec report>

  - >
    End with:
      - Total findings per axis
      - Worst issue in each axis
    Do NOT merge axes.

  # --- WHY TWO AXES ---
  - >
    Standards and Spec are independent:
      - Code can follow standards but implement the wrong thing.
      - Code can implement the right thing but violate standards.
    Keep axes separate.

examples:
  - user: "Review since main"
    agent: |
      ## Standards
      - React: Missing dependency array in useEffect in src/App.tsx.
      - TypeScript: Implicit any in utils/parse.ts.
      - CDK: IAM policy too broad (s3:*).
      - GitHub Actions: workflow has `permissions: write-all` instead of least privilege.

      ## Spec
      - Requirement "validate input schema" missing.
      - Scope creep: added caching not mentioned in issue #45.

      Summary: Standards: 4 findings (worst: IAM over-permission).
               Spec: 2 findings (worst: missing requirement).