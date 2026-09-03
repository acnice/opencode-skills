---
name: implement-feature
description: >
  Implement features using the strongest patterns adopted across popular
  OpenCode community skills: TypeScript strict mode, React best practices,
  AWS CDK architecture patterns, DynamoDB modeling, GitHub Actions CI,
  Jest/RTL testing, ESLint/Prettier, and pre-commit hooks. Includes mandatory
  review-before-commit. Use only after a plan exists.

risk: high
source: community
date_added: "2026-02-27"

capabilities:
  - implementation
  - test_authoring
  - infra_authoring
  - ci_cd_configuration
  - linting_setup
  - hooks_setup
  - local_review

triggers:
  - "implement this feature"
  - "build this"
  - "add endpoint"
  - "add page"
  - "cdk stack change"
---

instructions:
  # --- PRECONDITIONS ---
  - >
    Only run this skill when a plan exists (from @writing-plans). Follow tasks
    exactly, without skipping steps. No speculative generality.

  # --- STACK ASSUMPTIONS ---
  - >
    Assume:
      - React + TypeScript frontend
      - AWS CDK backend (Lambda, API Gateway)
      - DynamoDB data layer
      - GitHub Actions CI
      - Jest + React Testing Library
      - ESLint + Prettier
      - Husky or pre-commit hooks

  # --- IMPLEMENTATION WORKFLOW ---
  - >
    For each task:
      1. Write failing test (Jest/RTL/CDK assertions)
      2. Run tests → confirm FAIL
      3. Implement minimal code to pass tests
      4. Run full checks:
         - npm test
         - npm run lint
         - npm run typecheck
         - npm run format
      5. Run pre-commit hooks
      6. Run @code-review since last commit
      7. Fix critical issues
      8. Commit using conventional commits

  # --- TYPESCRIPT BEST PRACTICES (COMMUNITY STANDARD) ---
  - >
    Enforce:
      - strict mode
      - no implicit any
      - discriminated unions
      - avoid `as` casting
      - pure functions where possible
      - typed API responses
      - typed DynamoDB items

  # --- REACT BEST PRACTICES (COMMUNITY STANDARD) ---
  - >
    Enforce:
      - hooks rules (exhaustive deps)
      - component boundaries
      - state isolation
      - RTL tests for behavior, not implementation
      - avoid prop drilling (prefer context or composition)

  # --- CDK BEST PRACTICES (AWS COMMUNITY STANDARD) ---
  - >
    Enforce:
      - construct boundaries
      - least privilege IAM
      - safe RemovalPolicy
      - DynamoDB modeling (PK/SK, GSIs)
      - environment separation
      - avoid inline policies unless necessary

  # --- GITHUB ACTIONS BEST PRACTICES (COMMUNITY STANDARD) ---
  - >
    Enforce:
      - least-privilege permissions
      - OIDC for AWS
      - test/lint/typecheck on PR
      - fail on any error
      - avoid secrets in env

  # --- PRE-COMMIT HOOKS (COMMUNITY STANDARD) ---
  - >
    Enforce:
      - lint-staged
      - tests on staged files
      - typecheck on staged files
      - formatting on staged files

  # --- LOCAL REVIEW BEFORE COMMIT ---
  - >
    Mandatory:
      - Run @code-review with fixed point = last commit
      - Block commit if critical issues found

examples:
  - user: "Implement the login feature"
    agent: |
      Using implement-feature skill to implement the login feature.

      Task 1: Backend login Lambda
      - Write failing Jest test
      - Run tests (expect FAIL)
      - Implement minimal Lambda handler
      - Run lint/type/test
      - Run pre-commit hooks
      - Run @code-review since last commit
      - Commit with "feat: add login Lambda"