---
name: make-plan
description: >
  Create comprehensive, implementation-ready plans for multi-step engineering
  tasks. Use when a spec or requirements exist and code should not be touched
  yet. Produces deterministic, bite-sized, TDD-driven tasks with exact file
  paths, commands, and expected outputs. Designed for engineers with minimal
  context about the codebase.

risk: critical
source: community
date_added: "2026-02-27"

capabilities:
  - planning
  - decomposition
  - architecture_mapping
  - tdd_workflow
  - file_path_resolution
  - spec_alignment
  - execution_handoff

triggers:
  - "make a plan"
  - "write a plan"
  - "implementation plan"
  - "how do I build this"
  - "plan this feature"
---

instructions:
  # --- ANNOUNCEMENT ---
  - >
    At the start of every plan, announce:
    "Using writing-plans skill to generate the implementation plan."

  # --- WORKTREE REQUIREMENT ---
  - >
    Plans must assume execution inside a dedicated worktree created by
    @brainstorming. All paths must be relative to that worktree.

  # --- PLAN DOCUMENT HEADER ---
  - >
    Every plan MUST begin with:

    ```markdown
    # [Feature Name] Implementation Plan

    **Goal:** [One sentence describing what this builds]

    **Architecture:** [2-3 sentences describing the approach]

    **Tech Stack:** [React, TypeScript, AWS CDK, AWS Lambda, GitHub Actions, etc.]

    ---
    ```

  # --- PLAN STRUCTURE ---
  - >
    Plans must be decomposed into bite-sized tasks. Each task is 2–5 minutes of
    work and contains EXACT file paths, code, commands, and expected outputs.

  - >
    Each task MUST follow this structure:

    ```markdown
    ### Task N: [Component or Feature Name]

    **Files:**
    - Create: `exact/path/to/new-file.ts`
    - Modify: `exact/path/to/existing-file.ts:123-145`
    - Test: `tests/exact/path/to/test-file.test.ts`

    **Step 1: Write the failing test**
    ```ts
    test("specific behavior", () => {
      const result = fn(input)
      expect(result).toEqual(expected)
    })
    ```

    **Step 2: Run test to verify it fails**
    Run: `npm test -- tests/path/test-file.test.ts`
    Expected: FAIL with "fn is not defined"

    **Step 3: Write minimal implementation**
    ```ts
    export function fn(input: InputType): OutputType {
      return expected
    }
    ```

    **Step 4: Run test to verify it passes**
    Run: `npm test -- tests/path/test-file.test.ts`
    Expected: PASS

    **Step 5: Commit**
    ```bash
    git add tests/path/test-file.test.ts src/path/file.ts
    git commit -m "feat: implement specific behavior"
    ```
    ```

  # --- ENGINEERING PRINCIPLES ---
  - >
    Plans must enforce:
      - TDD (test first, minimal implementation)
      - DRY (no repetition in tasks)
      - YAGNI (no speculative generality)
      - Frequent commits (every task ends with a commit)
      - Deterministic steps (no vague instructions)
      - Exact file paths (never "add file somewhere")
      - Complete code snippets (never "add validation")

  # --- SPEC ALIGNMENT ---
  - >
    Plans must map every requirement in the spec to one or more tasks. If a
    requirement is ambiguous, ask the user before continuing.

  # --- TECH STACK AWARENESS ---
  - >
    Plans must incorporate stack-specific best practices:
      - React: component boundaries, hooks rules, state isolation
      - TypeScript: strict types, discriminated unions, no implicit any
      - CDK: construct boundaries, least privilege IAM, environment separation
      - AWS: Well-Architected (security, reliability, cost, performance)
      - GitHub Actions: least privilege permissions, OIDC, workflow hardening

  # --- SAVING PLANS ---
  - >
    Save the final plan to:
      docs/plans/YYYY-MM-DD-<feature-name>.md
  - >
    Ensure the docs/plans directory exists before saving. Create it if needed:
    `mkdir -p docs/plans`
  - >
    Use ISO 8601 date format (YYYY-MM-DD) for consistent sorting.
  - >
    Include a plan index file at `docs/plans/README.md` listing all plans with dates and status.

  # --- EXECUTION HANDOFF ---
  - >
    After saving, offer two execution modes:

    **"Plan complete and saved to `docs/plans/<filename>.md`. Choose execution mode:"**

    **1. Guided Execution (this session)**  
    - Use @executing-plans  
    - Execute tasks sequentially  
    - Review after each task  

    **2. Parallel Session (new tab)**  
    - User opens a new session in the worktree  
    - Use @executing-plans  
    - Batch execution with checkpoints

  # --- INDUSTRY STANDARDS & BEST PRACTICES ---
  - >
    Incorporate these standards to improve plan quality:
    - **RFC/ADR Format**: Document architectural decisions with context, decision, and consequences
    - **Risk Assessment**: Add a "Risks & Mitigations" section per task (technical, security, operational)
    - **Estimation**: Use story points or t-shirt sizing (XS/S/M/L/XL) for each task
    - **Dependency Graph**: Map task dependencies explicitly; visualize with Mermaid if complex
    - **Rollback Plan**: Every task must specify how to revert if it breaks something
    - **Security/Privacy**: Note data handling, auth boundaries, encryption requirements per task
    - **Observability**: Define metrics, logs, traces needed for each feature
    - **Definition of Done**: Explicit checklist per task (tests pass, lint clean, docs updated, reviewed)
    - **Code Review Checklist**: Include security, performance, accessibility, backward compatibility
    - **Capacity Planning**: Note resource requirements (CPU, memory, DB connections, API quotas)

  # --- LIMITATIONS ---
  - >
    Use this skill only when:
      - A spec or requirements exist
      - The task is multi-step
      - Code should not be touched yet

  - >
    Stop and ask for clarification if:
      - Required inputs are missing
      - Success criteria unclear
      - Safety boundaries unknown

examples:
  - user: "Write a plan for adding a new /login route"
    agent: |
      Using writing-plans skill to generate the implementation plan.

      # Login Route Implementation Plan

      **Goal:** Add a secure login route backed by AWS Lambda.

      **Architecture:** React form → API Gateway → Lambda → DynamoDB user table.

      **Tech Stack:** React, TypeScript, AWS CDK, Lambda, Jest.

      ---
      (...tasks follow...)