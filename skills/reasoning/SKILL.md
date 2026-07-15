---
name: reasoning
description: Use when the user asks for a reasoning session.
---

# reasoning

Use this skill before implementing any user-provided coding goal.

## Purpose

Prepare the current Codex context for fast implementation by understanding the existing code, identifying uncertainty, asking targeted questions, and keeping all implementation knowledge inside the current conversation context.

Do not create a separate spec file unless the user explicitly asks for one.

## Core behavior

When the user gives an implementation goal:

1. Restate the goal briefly.
2. Inspect the existing repository before asking broad questions.
3. Search for relevant files, symbols, routes, tests, configs, types, schemas, and existing patterns.
4. Read enough surrounding code to understand how the current implementation works.
5. Build a compact mental model of:
   - affected modules
   - current data flow
   - existing abstractions
   - conventions already used
   - tests or validation points
   - risks and unknowns
6. Ask questions whenever anything is missing, ambiguous, or uncertain.
7. For each question, present possible implementation approaches and a recommendation.
8. Continue inspecting code and asking follow-up questions until implementation can proceed confidently.
9. Before coding, output a compact implementation-readiness brief.
10. Then implement inside the same context window.

## Question policy

Do not ask questions that can be answered reliably by inspecting the repository.

Ask questions for any uncertainty that could affect implementation, including small details.

Ask about:

- expected behavior
- edge cases
- UI/API contracts
- data shape
- migrations
- backwards compatibility
- permissions
- feature flags
- naming
- architectural direction
- test expectations
- unclear existing behavior

## Question format

When asking implementation questions, do not ask bare questions.

Use this format:

### Decision needed: <short title>

**Current understanding**  
<brief summary based on code inspection>

**Possible approaches**

1. <approach A> — <pros/cons>
2. <approach B> — <pros/cons>
3. <approach C, if useful> — <pros/cons>

**Recommendation**  
<preferred approach and why>

**Question**  
<specific decision or missing detail needed from the user>

Keep each question round focused. Prefer fewer high-signal questions over long questionnaires.

## Readiness threshold

Do not start implementation until:

- the goal is clear
- the current implementation is understood
- likely affected files are known
- important edge cases are resolved
- validation strategy is known
- user decisions are captured
- remaining assumptions are minor and explicit

## Final pre-implementation brief

Before coding, output this compact YAML block:

```yaml
implementation_readiness:
  goal: ...
  current_implementation: ...
  files_likely_to_change:
    - ...
  confirmed_decisions:
    - ...
  remaining_assumptions:
    - ...
  implementation_plan:
    - ...
  validation_plan:
    - ...
```
