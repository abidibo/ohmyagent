---
name: spec
description: Spec-driven development skill. Supports subcommands — `/spec` or `/spec create` to write a new spec, `/spec status` to show progress across all specs in the project. Use when the user wants to write a spec or check spec progress. Especially when invoked directly.
---

# Spec skill

Any response that skips a required step is incorrect. Do not optimize for speed over compliance.

## Invocation

This skill supports subcommands. Check the invocation before doing anything else:

- `/spec` or `/spec create` — run the full spec creation workflow (default).
- `/spec status` — show a summary table of all specs in the project.
- `/spec status <feature-name>` — show detailed step-by-step progress for a single spec.

If no argument is given, default to `create`.

---

## Command: status

### `/spec status` — all specs

1. Search for all `progress.yaml` files under `specs/` in the current project root:
   ```bash
   find specs -name "progress.yaml" | sort
   ```
2. Read each file and extract: `feature`, `status`, total task count, completed task count.
3. Present a formatted summary table:

   ```
   Feature              Progress         Tasks      Status
   ──────────────────── ──────────────── ────────── ────────────
   user-auth            ████████░░░░     6 / 10     in_progress
   payment-flow         ░░░░░░░░░░░░     0 / 5      pending
   search-index         ████████████    12 / 12     completed
   ```

   - Progress bar: 12 chars wide. Fill `█` proportional to completed/total tasks, remaining with `░`.
   - Sort: `in_progress` first, then `pending`, then `completed`.
   - If `specs/` does not exist or contains no `progress.yaml` files, print:
     "No progress files found. Run `/spec` to create a spec, then opt in to progress tracking."

4. After the table, print a one-line summary:
   ```
   3 specs — 1 completed, 1 in progress, 1 pending
   ```

### `/spec status <feature-name>` — single spec detail

1. Look for `specs/<feature-name>/progress.yaml`. If not found, print:
   "No progress file found for '<feature-name>'. Check the feature name or run `/spec status` to list all specs."
2. Read the file and present a detailed breakdown — one section per step, with all its tasks listed:

   ```
   user-auth — in_progress — 6 / 10 tasks completed

   Step 1: Add JWT middleware  [completed]
     ✔  1.1  Install and configure jwt library
     ✔  1.2  Write middleware function that validates token from Authorization header
     ✔  1.3  Attach decoded user payload to request context

   Step 2: Protect routes  [in_progress]
     ✔  2.1  List all routes requiring authentication
     ✗  2.2  Apply middleware to protected route group
     ✗  2.3  Return 401 on missing or invalid token

   Step 3: Write tests  [pending]
     ✗  3.1  Test valid token is accepted
     ✗  3.2  Test expired token is rejected
     ✗  3.3  Test missing token returns 401
   ```

   - `✔` for `completed` tasks, `✗` for `pending` or `in_progress`.
   - Step status shown inline: `[completed]`, `[in_progress]`, `[pending]`.
   - Print a one-line summary at the top: `<feature> — <status> — X / Y tasks completed`.

Do not enter the creation workflow. Exit after printing the status.

---

## Command: create (default)

When invoked as `/spec` or `/spec create`:

Any response that skips a required step is incorrect.

## When to use

Use when the user wants to produce a specification document for a feature or new piece of work to implement. The output is a `spec.md` file that a senior developer or AI agent can use directly as an implementation guide.

## Output

Determine the output path before starting:

1. Check if a `specs/` folder exists in the root of the project.
2. If it exists, inspect its structure and follow the existing pattern exactly.
3. If it does not exist, create: `specs/<feature-name>/spec.md`

The `<feature-name>` is derived from the feature title provided by the user (kebab-case).

### Optional: progress.yaml

If the user opts in (asked at the end of the workflow), generate `specs/<feature-name>/progress.yaml` alongside the spec. This file expands each spec step into nearly atomic tasks and tracks their completion status. It is designed to be read and updated by the implement skill during implementation.

```yaml
feature: <feature-name>
spec: specs/<feature-name>/spec.md
created: <ISO 8601 date>
status: pending          # pending | in_progress | completed
started_at: ~            # set when first step is started
completed_at: ~          # set when all steps are completed

steps:
  - id: 1
    title: "Step title from spec"
    status: pending      # pending | in_progress | completed
    started_at: ~        # set when step begins
    completed_at: ~      # set when step is marked completed
    tasks:
      - id: "1.1"
        description: "Install and configure the X library"
        status: pending  # pending | in_progress | completed
        started_at: ~
        completed_at: ~
      - id: "1.2"
        description: "Write the Y function in src/module.py"
        status: pending
        started_at: ~
        completed_at: ~
  - id: 2
    title: "Another step"
    status: pending
    started_at: ~
    completed_at: ~
    tasks:
      - id: "2.1"
        description: "..."
        status: pending
```

**Task granularity rules:**
- Each task must be a single, independently actionable unit of work (one function, one file, one config change).
- Do not copy step titles as tasks. Derive tasks from the step's actual content and subtasks.
- Tasks should be specific enough that a developer can tick them off without ambiguity.

## Spec file structure

Every spec file must contain the following sections, in order:

```
# <Feature Name>

## Summary
One or two sentences describing what this is.

## Problem Statement
What problem does this solve? Why does it need to be solved? Be precise.

## Solution Overview
High-level description of the chosen solution and why it was selected.

## Technical Context
Relevant existing files, modules, APIs, patterns, and conventions in the codebase that are directly involved. Include real file paths and function/class names discovered during codebase exploration.

## Implementation Steps
Ordered, atomic, granular steps. Each step must be independently understandable and implementable.
For each step:
- Clear title
- What to do and why
- Code or pseudocode (depth depends on detail level)
- Any relevant file paths or interfaces

## Visual Aids
Tables, diagrams, flow charts, data models — only when they genuinely clarify the design.
If an HTML visual was generated during the session, reference or embed it here.

## Acceptance Criteria
A checklist of observable, verifiable conditions that confirm the feature is correctly and completely implemented.

## Alternatives Considered
[Include only when real tradeoffs were surfaced. Omit entirely otherwise.]
For each alternative: what it was, why it was considered, why it was rejected.
```

The depth of each section scales with the detail level chosen by the user:
- `low` — brief descriptions, pseudocode only, key decisions noted
- `medium` — step-by-step descriptions, partial code examples, key file references
- `high` — full code examples, all file paths, function signatures, edge cases, complete decision rationale

## Workflow

You MUST follow this workflow strictly. Do not skip or reorder steps.

1. **Kickoff** — Ask the user two things only:
   - *"Describe what you want to build and why. Share any relevant links, documents, or code."*
   - *"What detail level do you want for this spec: low, medium, or high?"*

2. **Codebase exploration** — Before asking clarifying questions, explore the codebase to understand the relevant context. Declare what you are reading and why. Ask the user if there are specific areas to focus on. Read files, git logs, existing patterns — anything that grounds your understanding.

3. **Clarify until the problem is unambiguous** — Ask clarifying questions one at a time. Do not proceed to spec writing until you are fully confident about:
   - The exact problem being solved
   - The boundaries of the solution (in scope / out of scope)
   - The success criteria
   - Any constraints or dependencies
   If uncertain about anything, ask. Never assume.

4. **Propose a solution approach** — Present 2 or 3 possible approaches with a clear recommendation and rationale. Ask the user to choose or propose something different.

5. **Visual aid (when helpful)** — If a diagram, flow, or data model would clarify the design or the implementation plan, generate a static HTML file in a tmp directory and open it with `xdg-open`. Use tables, flowcharts, component diagrams, data models — whatever best represents the concept. Lean toward generating visuals when in doubt. All generated HTML must follow `STYLE.md` exactly.

6. **Present the implementation steps** — Break the solution into ordered, atomic, granular steps. Display them in a table for review:

   | # | Step | Description | Scope |
   |---|------|-------------|-------|
   | 1 | ... | ... | ... |

   Ask the user to review. Allow merging, splitting, reordering, adding, or removing steps before proceeding.

7. **Write the spec** — Once steps are approved, write the full `spec.md` file following the structure above. Apply the chosen detail level throughout.

8. **Review** — Ask the user to review the spec. Collect corrections and revise until the user is satisfied.

## Process Flow

```yaml
workflow:
  name: spec_creation
  description: >
    The agent must follow this workflow strictly for any spec creation task.
    It must not proceed to writing the spec until the problem is fully understood
    and the implementation steps are approved by the user.

  global_rules:
    - The target reader is always a senior developer or an AI agent acting as implementor. Be precise, unambiguous, and technical.
    - Ask clarifying questions one at a time.
    - Never proceed to spec writing until the problem is fully understood.
    - Prefer visual representation — tables, diagrams, flowcharts. When in doubt, generate the HTML visual.
    - Propose 2 to 3 solution approaches with a recommendation before settling on one.
    - Implementation steps must be atomic and granular enough to be implemented independently.
    - Include acceptance criteria in every spec.
    - Include alternatives only when real tradeoffs were surfaced — omit otherwise.
    - Explore the codebase proactively but declare what you are reading and why.

  states:
    - id: kickoff
      title: Kickoff
      objective: Collect the feature description, context, and desired detail level.
      agent_actions:
        - Ask the user to describe what they want to build and why.
        - Ask the user to share relevant links, documents, or code.
        - Ask the user for the detail level (low / medium / high).
      exit_condition:
        - Feature description and detail level are provided.

    - id: explore
      title: Codebase exploration
      objective: Ground understanding in the actual codebase before asking questions.
      agent_actions:
        - Declare what files or areas you intend to read and why.
        - Ask the user if there are specific areas to focus on.
        - Read relevant files, patterns, existing modules, git history.
        - Identify project conventions (naming, architecture, style, testing patterns) — these become the primary best practice reference for the spec.
      exit_condition:
        - Relevant codebase context is understood.

    - id: clarify
      title: Clarify until unambiguous
      objective: Achieve full, unambiguous understanding of the problem.
      agent_actions:
        - Ask one clarifying question at a time.
        - Continue until the problem, scope, success criteria, and constraints are clear.
      exit_condition:
        - Problem is fully understood with no material ambiguity remaining.

    - id: propose_approach
      title: Propose solution approaches
      objective: Align on the solution direction before planning steps.
      agent_actions:
        - Present 2 or 3 approaches.
        - State the recommended approach and explain why.
        - Ask the user to choose or suggest an alternative.
      exit_condition:
        - The user selects or approves an approach.

    - id: visual
      title: Visual aid (conditional)
      objective: Clarify design or implementation plan visually when it adds value.
      agent_actions:
        - Judge whether a visual would genuinely help. Lean toward yes.
        - If yes, generate a static HTML file with diagrams/tables/flows in a tmp dir.
          Follow STYLE.md exactly for all markup, tokens, CDN libraries, and aesthetics.
        - Open the file with xdg-open.
        - Ask the user if the visual correctly represents the design.
      exit_condition:
        - Visual generated and confirmed, or judged unnecessary.

    - id: plan_steps
      title: Plan implementation steps
      objective: Define and agree on the ordered, atomic implementation steps.
      agent_actions:
        - Break the solution into granular, atomic, ordered steps.
        - Present steps in a table for user review.
        - Allow the user to merge, split, reorder, add, or remove steps.
      exit_condition:
        - The user approves the implementation step plan.

    - id: write_spec
      title: Write the spec
      objective: Produce the spec.md file.
      agent_actions:
        - Determine the output path (follow existing specs/ pattern or create specs/<feature-name>/spec.md).
        - Write the full spec.md following the defined structure.
        - Apply the chosen detail level throughout.
      exit_condition:
        - spec.md is written.

    - id: review
      title: Review
      objective: Validate the spec with the user.
      agent_actions:
        - Ask the user to review the spec.
        - Collect corrections and revise until the user is satisfied.
      exit_condition:
        - The user is satisfied with the spec.

    - id: generate_progress
      title: Generate progress file (optional)
      objective: Optionally produce a progress.yaml alongside the spec to track implementation.
      agent_actions:
        - Ask the user: "Would you like a progress.yaml file to track implementation of this spec?"
        - If declined, skip and end.
        - If accepted:
            - For each implementation step in the spec, expand it into nearly atomic tasks.
              Each task must be independently actionable — a single function, file, or config change.
              Do not copy step titles verbatim; derive granular tasks from the step's content.
            - Write progress.yaml in the same directory as spec.md.
            - Format (YAML):
                feature: <feature-name>
                spec: <relative path to spec.md>
                created: <today's date ISO 8601>
                status: pending          # pending | in_progress | completed
                started_at: ~            # set when first step is started
                completed_at: ~          # set when all steps are completed
                steps:
                  - id: <step number>
                    title: <step title>
                    status: pending      # pending | in_progress | completed
                    started_at: ~        # set when step begins
                    completed_at: ~      # set when step is marked completed
                    tasks:
                      - id: <step>.<task>
                        description: <atomic task description>
                        status: pending  # pending | in_progress | completed
                        started_at: ~    # set when task begins
                        completed_at: ~  # set when task is marked completed
      exit_condition:
        - User declines, OR progress.yaml is written.

  transitions:
    - from: kickoff
      to: explore
      when: description_and_detail_level_provided

    - from: explore
      to: clarify
      when: codebase_context_understood

    - from: clarify
      to: clarify
      when: ambiguity_remains

    - from: clarify
      to: propose_approach
      when: problem_fully_understood

    - from: propose_approach
      to: propose_approach
      when: user_requests_different_options

    - from: propose_approach
      to: visual
      when: approach_selected

    - from: visual
      to: plan_steps
      when: visual_confirmed_or_skipped

    - from: plan_steps
      to: plan_steps
      when: user_requests_step_changes

    - from: plan_steps
      to: write_spec
      when: steps_approved

    - from: write_spec
      to: review
      when: spec_written

    - from: review
      to: write_spec
      when: user_requests_corrections

    - from: review
      to: generate_progress
      when: user_is_satisfied

    - from: generate_progress
      to: end
      when: user_declines_or_progress_file_written
```

## Key principles

- **Precision over brevity** — the spec will be read by a senior dev or an AI agent. Ambiguity is a bug.
- **Grounded in reality** — reference real file paths, real function names, real patterns from the codebase.
- **Atomic steps** — each implementation step must be small enough to implement and verify independently.
- **Visual when it helps** — a table or diagram communicates faster than three paragraphs. Prefer it.
- **No filler** — every section must earn its place. Omit alternatives, decisions, or context that adds no value.
- **Best practices, always** — every implementation step and design decision must reflect best practices. Priority order:
  1. **Project conventions first** — during codebase exploration, identify existing patterns, naming conventions, architecture decisions, and coding style. The spec must align with them.
  2. **Ecosystem best practices second** — where the project has no established pattern, apply industry best practices for the detected language, framework, and architecture (e.g., SOLID, security, performance, testability).
  3. **Justify any deviation** — if a step deviates from project conventions or ecosystem best practices, the spec must explicitly state why and what the tradeoff is.

## Mandatory rules

- Do not write the spec until steps 1–4 (kickoff, explore, clarify, propose approach) have been completed.
- Do not proceed past clarification while any material ambiguity about the problem remains.
- Always present implementation steps for user approval before writing the spec.
- Every step and decision in the spec must reflect best practices: project conventions first, ecosystem best practices second. Any deviation must be explicitly justified.
- If the user asks to skip the workflow, still complete at minimum: problem understanding + step approval.
- If you cannot follow this workflow, say so and stop instead of proceeding.

## Execution gate

Writing the spec is forbidden until:

1. The problem is fully understood with no ambiguity remaining.
2. A solution approach has been selected by the user.
3. The implementation steps have been reviewed and approved by the user.

!IMPORTANT: The spec reflects a shared understanding between the user and the agent — never write it unilaterally.
