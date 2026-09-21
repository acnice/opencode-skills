---
name: frontend-design
description: Produce thoughtful, high-quality frontend design work focused on UX clarity, layout hierarchy, spacing, interaction patterns, and design rationale. Use when designing or improving UI, layouts, dashboards, flows, or any user-facing frontend.
---

triggers:
  - "design this UI"
  - "design the interface"
  - "improve this layout"
  - "frontend design"
  - "make this look better"
  - "design the dashboard"
  - "design the flow"
  - "clean up this UI"
  - "fix the spacing"
  - "improve the UX"

instructions:
  # --- CORE BEHAVIOR ---
  - >
    Think like a product designer before writing any code.
    Prioritize clarity, simplicity, and user comprehension.
    Explain the design intent before implementing UI.
  - >
    Use consistent spacing, alignment, and hierarchy.
    Choose components deliberately, not automatically.
    Consider accessibility and inclusive design at every step.
  - >
    Every UI decision must be intentional, consistent, and grounded in strong design principles.
    No generic UI without rationale.

  # --- UX PRINCIPLES ---
  - >
    Start by describing the user's goal and context.
    Map the user flow step-by-step.
    Identify friction points and remove them.
  - >
    Ensure labels, tooltips, and messages are concise and human.
    Use clear hierarchy: primary actions, secondary actions, supporting info.
    Prefer predictable patterns over clever or unusual UI.

  # --- LAYOUT & VISUAL DESIGN ---
  - >
    Define the layout structure before coding (sections, columns, spacing).
    Maintain consistent vertical rhythm and spacing scale.
    Group related elements together; separate unrelated ones.
    Use alignment to create visual order.
  - >
    Ensure responsive behavior across breakpoints.
    Provide loading, empty, and error states for all components.

  # --- COMPONENT USAGE ---
  - >
    Use components intentionally based on their purpose.
    Avoid mixing patterns (e.g., modal vs drawer vs inline form).
    Ensure components follow the product's design system.
    Keep interactions simple and predictable.
    Provide clear feedback for all user actions.

  # --- ACCESSIBILITY ---
  - >
    Ensure keyboard navigation works.
    Provide meaningful focus states.
    Use semantic HTML structure.
    Avoid ambiguous icons without labels.
    Ensure color contrast meets accessibility standards.

  # --- OUTPUT RULES ---
  - >
    Before producing UI or code, always output:
      1. The design intent.
      2. The user flow.
      3. The layout and hierarchy.
      4. Validation against the design system.
    Only then produce the UI/code.

  # --- WHAT THIS SKILL AVOIDS ---
  - >
    No generic UI without rationale.
    No cluttered layouts or inconsistent spacing.
    No unexplained component choices.
    No filler text or AI-style verbosity.
    No inaccessible patterns.

  # --- FORMAT ---
  - >
    For each design task, format EXACTLY like this:

    Design intent: <one or two sentences on the user goal and why this design serves it>
    User flow: <numbered steps from entry point to completion>
    Layout & hierarchy: <sections, columns, spacing, primary/secondary actions>
    Design system check: <components and tokens reused, or deviations justified>
    UI/code: <the implementation>

  - >
    After producing the UI/code, briefly note any states handled (loading, empty, error)
    and any accessibility considerations applied.

  # --- EXAMPLE PROMPTS ---
  - "Use Frontend Design to design the inspection walkthrough UI."
  - "Use Frontend Design to create a clean layout for schedule creation."
  - "Use Frontend Design to improve the monthly audit dashboard."

  # --- GOAL ---
  - >
    Deliver frontend work that feels deliberate, polished, and professionally designed —
    with clear UX reasoning and consistent visual structure.
