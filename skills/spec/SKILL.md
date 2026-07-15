---
name: spec
description: Spec-driven development skill. Supports subcommands — `/spec` or `/spec create` to write a new spec, `/spec status` to show progress across all specs in the project. Use when the user wants to write a spec or check spec progress. Especially when invoked directly. Spec creation runs an interactive brainstorming workflow — themed decision rounds with options and recommendations, scaled to project size, with master + sub-spec support for multi-component systems.
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
Tables, zoomable diagrams, zoomable flow charts, data models — only when they genuinely clarify the design.
If an HTML visual was generated during the session, reference or embed it here.

## Failure Modes & Recovery
For each implementation step that involves external services, data mutations, concurrency, user input, or any operation that can fail:
- Identify the failure modes (network errors, timeouts, invalid state, partial writes, race conditions, etc.)
- Describe the chosen recovery strategy (retry with backoff, fallback, graceful degradation, fail-fast, rollback, idempotency guard, etc.)
- State what the user/system observes when the failure occurs (error message, default behavior, logged warning, etc.)

Scale depth to risk: a simple CRUD endpoint may need one line; a multi-step payment flow needs detailed per-step analysis.

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

For small, single-component features (a handful of steps, no new architecture), keep every gate but compress the ceremony: steps 4–7 may be bundled into one message — recommended approach, step table, and failure-mode defaults — closed by a single approval question. The gates exist to guarantee shared understanding, not to maximize round-trips.

1. **Kickoff** — Extract answers from the conversation first when the feature was already discussed; only ask what is missing:
   - *"Describe what you want to build and why. Share any relevant links, documents, or code."*
   - *"What detail level do you want for this spec: low, medium, or high?"*
   - For multi-component or greenfield projects, also settle spec scoping — single spec, master + sub-specs (see "Multi-spec projects"), or first-phase-only — and, when relevant, the project/feature name. Structured questions with a marked recommendation work well for these.

2. **Codebase exploration** — Before asking clarifying questions, explore the codebase to understand the relevant context. Declare what you are reading and why. Ask the user if there are specific areas to focus on. Read files, git logs, existing patterns — anything that grounds your understanding. When the feature depends on external systems reachable from this session (APIs, MCP servers, services), verify the load-bearing facts with cheap read-only calls now — real IDs, real payload shapes, whether a capability actually exists — and record in the spec what is live-verified versus assumed. A spec pinned to reality beats one pinned to documentation.

3. **Clarify through themed decision rounds** — Group related open decisions into themed rounds (context/deployment, data model, UX, integrations, runtime, …), 2–4 decisions per round, presented as structured questions (the AskUserQuestion tool when available, otherwise a compact numbered list). One topic gets settled, then you move on — this keeps decision throughput high without overwhelming the user. For each question:
   - Offer 2–4 concrete options with trade-offs stated in the option descriptions; put your recommendation first, marked "(Recommended)", with the rationale. Users mostly confirm recommendations — the value is that every choice becomes *visible and decided now* instead of buried in the spec later.
   - Use multi-select when choices are not mutually exclusive.
   - Treat free-text answers as first-class design input — they often carry the most shaping decisions. If an answer contains a question, answer it concretely before proceeding; if it redirects the design, adapt and confirm the consequence in your next message.
   - Open each round by recapping what the previous round locked ("Locked: X, Y, Z") and keep a running decision ledger — it prevents relitigating and later becomes a section of the visual and the spec.
   Scale the number of rounds to the scope: a small feature needs one round or none; a greenfield multi-component system may need many. Do not run ceremony the project doesn't need.
   Do not proceed to spec writing until you are fully confident about:
   - The exact problem being solved
   - The boundaries of the solution (in scope / out of scope)
   - The success criteria
   - Any constraints or dependencies
   If uncertain about anything material, ask. Never assume.

4. **Propose a solution approach** — Present 2 or 3 possible approaches with a clear recommendation and rationale. Ask the user to choose or propose something different.

5. **Visual aid (when helpful)** — If a diagram, flow, or data model would clarify the design or the implementation plan, generate a static HTML file in a tmp directory and open it with `xdg-open`. Use tables, flowcharts, component diagrams, data models — whatever best represents the concept. Generate one when the design has structure a diagram or table communicates faster than prose; skip it for single-file changes. After a long brainstorming session, include the decision ledger as a section of the visual so the user can audit everything agreed in one place. All generated HTML must follow the STYLE.md section at the bottom of this skill file exactly. If no display is available (headless/remote session), save the file and give the user its path instead of running `xdg-open`.

6. **Present the implementation steps** — Break the solution into ordered, atomic, granular steps. Display them in a table for review:

   | # | Step | Description | Scope |
   |---|------|-------------|-------|
   | 1 | ... | ... | ... |

   Ask the user to review. Allow merging, splitting, reordering, adding, or removing steps before proceeding.

7. **Failure mode review** — Once steps are approved, review each step for potential failure modes. For steps involving external services, data mutations, concurrency, user input, or any operation that can fail:
   - Identify what can go wrong (network errors, timeouts, invalid state, partial writes, race conditions, etc.)
   - Propose a recovery strategy for each (retry, fallback, graceful degradation, fail-fast, rollback, etc.)
   - Present all failure modes with proposed strategies as a table, but escalate only the genuinely user-owned calls (data-loss policy, cost policy, what the user sees on failure) as a structured question round — for the mechanical rows, a proposed default with a clear rationale is enough. Do not assume on the escalated ones.
   - Scale depth to complexity: skip this for purely structural or trivial steps; be thorough for anything that touches I/O, state, or external systems.

8. **Write the spec** — Once steps and failure modes are approved, write the full `spec.md` file following the structure above. Apply the chosen detail level throughout. For multi-component projects using master + sub-specs, write and review the master (with its numbered contracts) first, then fan out sub-specs in parallel and reconcile — see "Multi-spec projects".

9. **Review** — Ask the user to review the spec. Collect corrections and revise until the user is satisfied.

## Multi-spec projects (master + sub-specs)

A single `spec.md` stops working when the feature is really a system — several independently implementable components (e.g. a schema, a renderer, an engine, a GUI) that each need their own steps, failure modes, and progress tracking. Offer this structure at kickoff whenever you can see 3+ such components or the step plan would clearly exceed ~15 steps; let the user choose between one big spec, master + sub-specs, or specing only the first phase.

When master + sub-specs is chosen:

1. **Master first, contracts first.** `specs/<feature>/spec.md` holds the architecture and — critically — every cross-component contract (data formats, interfaces, storage layouts, config, CLI surface), numbered (C1, C2, …) so sub-specs can cite them. Contracts are the expensive-to-change surface: have the user review them *before* any sub-spec exists. Sub-specs reference contracts, never redefine them — that is what keeps N documents coherent.
2. **Fan out sub-specs in parallel.** Draft each `specs/<feature>-<component>/spec.md` with a subagent. Prefer agents that inherit the full conversation (all decisions and context travel for free); otherwise include the decision ledger and the master spec path in the prompt. Each writer gets: its exact scope (which master steps), the spec file structure to follow, the binding-contracts rule, and a reporting duty — return a short summary of judgment calls made beyond the master plus any contract ambiguity hit. The flags are the point: writers decide locally so drafting stays parallel, but every decision surfaces for reconciliation.
3. **Consistency pass.** When writers return, reconcile before showing the user: adopt flagged deltas into the master (amend the contracts), fix cross-references between coupled specs (APIs one spec assumes another provides), and resolve conflicts between writers — pick the resolution that preserves the stronger invariant and patch the losing spec. Verify couplings with targeted greps over the spec files rather than rereading everything.
4. **Escalate only real judgment calls.** Most flags are mechanical adoptions. The few that genuinely belong to the user (a dropped feature resurfacing, a policy choice) go back as one final structured-question round together with the review request.
5. **Progress files per sub-spec.** On opt-in, each sub-spec gets its own `progress.yaml` — send the request back to each sub-spec's author agent when possible, since it still holds its spec in context — and the master `progress.yaml` tracks only master-level steps. `/spec status` then shows per-component progress. When the master defines a global step numbering, keep it inside the sub-spec progress files so the two views line up.

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
    - Clarify via themed decision rounds - 2 to 4 related decisions per round as structured questions with concrete options, trade-offs, and a marked recommendation. Scale the number of rounds to project scope.
    - Maintain a running decision ledger. Recap what each round locked before opening the next. Never relitigate a locked decision unless the user reopens it.
    - Treat free-text answers as design input. Answer questions embedded in them before proceeding.
    - Verify load-bearing external facts with cheap read-only calls during exploration. Label spec data as live-verified vs assumed.
    - Never proceed to spec writing until the problem is fully understood.
    - Generate the HTML visual when the design has structure a diagram or table shows faster than prose (architecture, data models, a long decision ledger); skip it for single-file changes.
    - Propose 2 to 3 solution approaches with a recommendation before settling on one.
    - Implementation steps must be atomic and granular enough to be implemented independently.
    - Include acceptance criteria in every spec.
    - Review each implementation step for failure modes. For steps involving I/O, state mutations, concurrency, or external dependencies, identify what can go wrong and propose recovery strategies. Escalate only user-owned calls (data loss, cost, user-visible failure behavior) as questions; propose defaults with rationale for the rest. Scale depth to risk.
    - Include alternatives only when real tradeoffs were surfaced — omit otherwise.
    - Explore the codebase proactively but declare what you are reading and why.

  states:
    - id: kickoff
      title: Kickoff
      objective: Collect the feature description, context, and desired detail level.
      agent_actions:
        - Extract the feature description and any already-made decisions from the conversation first; only ask for what is missing.
        - Ask the user to describe what they want to build and why (if not already provided), and to share relevant links, documents, or code.
        - Ask the user for the detail level (low / medium / high).
        - For multi-component or greenfield projects, settle spec scoping (single spec vs master + sub-specs vs first-phase-only) with a recommendation.
      exit_condition:
        - Feature description and detail level are provided (and, for multi-component projects, the spec scoping decision).

    - id: explore
      title: Codebase exploration
      objective: Ground understanding in the actual codebase before asking questions.
      agent_actions:
        - Declare what files or areas you intend to read and why.
        - Ask the user if there are specific areas to focus on.
        - Read relevant files, patterns, existing modules, git history.
        - Verify load-bearing facts about reachable external systems (APIs, MCP servers, services) with cheap read-only calls; note verified values for the spec.
        - Identify project conventions (naming, architecture, style, testing patterns) — these become the primary best practice reference for the spec.
      exit_condition:
        - Relevant codebase context is understood.

    - id: clarify
      title: Clarify until unambiguous
      objective: Achieve full, unambiguous understanding of the problem.
      agent_actions:
        - Group open decisions into themed rounds (2-4 per round) presented as structured questions with options, trade-offs, and a marked recommendation.
        - Recap locked decisions at the start of each round; maintain the decision ledger.
        - Answer questions embedded in free-text replies concretely; adapt the design and confirm consequences.
        - Scale rounds to scope; continue until the problem, scope, success criteria, and constraints are clear.
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
        - Judge whether a visual would genuinely help - structure a diagram or table shows faster than prose. Skip for single-file changes.
        - If yes, generate a static HTML file with diagrams/tables/flows in a tmp dir.
          Follow the STYLE.md section at the bottom of this file exactly for all markup, tokens, CDN libraries, and aesthetics.
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

    - id: failure_modes
      title: Failure mode review
      objective: Identify failure modes and agree on recovery strategies before writing the spec.
      agent_actions:
        - For each approved step, assess whether it involves external services, data mutations, concurrency, user input, or any operation that can fail.
        - For steps with failure potential, identify specific failure modes (network errors, timeouts, invalid state, partial writes, race conditions, permission errors, etc.).
        - Propose a recovery strategy for each failure mode (retry with backoff, fallback, graceful degradation, fail-fast, rollback, idempotency guard, circuit breaker, etc.).
        - Present all failure modes with proposed strategies as a table; escalate only genuinely user-owned calls (data-loss policy, cost policy, user-visible failure behavior) as structured questions — proposed defaults with rationale suffice for mechanical rows. Do not assume on the escalated ones.
        - Scale depth to risk — skip trivial or purely structural steps; be thorough for anything that touches I/O, state, or external systems.
        - If no step has meaningful failure modes, state that explicitly and move on.
      exit_condition:
        - User approves failure mode handling for all relevant steps, or confirms no failure modes need addressing.

    - id: write_spec
      title: Write the spec
      objective: Produce the spec.md file.
      agent_actions:
        - Determine the output path (follow existing specs/ pattern or create specs/<feature-name>/spec.md).
        - Write the full spec.md following the defined structure.
        - Apply the chosen detail level throughout.
        - For master + sub-spec projects - write the master (architecture + numbered contracts) first and review it with the user; then draft sub-specs in parallel via subagents that inherit context and report judgment calls and contract ambiguities; then run the consistency pass (adopt deltas into the master, fix cross-references, resolve writer conflicts) before presenting.
      exit_condition:
        - spec.md is written (and, for multi-spec projects, sub-specs are written and reconciled).

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
            - Write progress.yaml in the same directory as spec.md. For master + sub-spec projects, write one per sub-spec (delegate to each sub-spec's author agent when it still holds its spec in context) plus a master file covering only master-level steps.
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
      to: failure_modes
      when: steps_approved

    - from: failure_modes
      to: failure_modes
      when: user_requests_changes_to_failure_handling

    - from: failure_modes
      to: write_spec
      when: failure_modes_approved_or_none_applicable

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

- Do not write the spec until steps 1–4 (kickoff, explore, clarify, propose approach) are complete and the step plan (step 6) and failure modes (step 7) are approved — the same conditions as the Execution gate.
- Do not proceed past clarification while any material ambiguity about the problem remains.
- Always present implementation steps for user approval before writing the spec.
- Every step and decision in the spec must reflect best practices: project conventions first, ecosystem best practices second. Any deviation must be explicitly justified.
- If the user asks to skip the workflow, still complete at minimum: problem understanding + step approval. Failure modes may then be handled as proposed defaults inside that same approval message — the execution gate's failure-mode item is satisfied by the bundled approval.
- If you cannot follow this workflow, say so and stop instead of proceeding.

## Execution gate

Writing the spec is forbidden until:

1. The problem is fully understood with no ambiguity remaining.
2. A solution approach has been selected by the user.
3. The implementation steps have been reviewed and approved by the user.
4. Failure modes have been identified and recovery strategies approved by the user (or explicitly confirmed as not applicable).

!IMPORTANT: The spec reflects a shared understanding between the user and the agent — never write it unilaterally.

---

## STYLE.md — Visual Style Guide

This file defines the visual style for all HTML artifacts produced during the spec workflow (diagrams, flowcharts, data models, tables, etc.).

This should be a dark theme modern elegant cool frontend application.

---

## Format

Always produce a **single, self-contained static HTML file**. All CSS and JS must be loaded from CDN — no build steps, no separate files, no inline `<style>` blobs for layout (use Tailwind utility classes instead).

---

## CDN libraries

Always include these in `<head>` / end of `<body>` as needed:

```html
<!-- Tailwind CSS -->
<script src="https://cdn.tailwindcss.com"></script>

<!-- Mermaid (diagrams, flowcharts) -->
<script src="https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.min.js"></script>

<!-- Alpine.js (lightweight interactivity, collapsible sections, tabs) — add only when needed -->
<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3/dist/cdn.min.js"></script>
```

Add other libraries only when genuinely required by the content (e.g. a chart library for data visualizations). Never include libraries that are not used.

---

## Tailwind configuration

Extend the default config to register the design tokens used throughout:

```html
<script>
  tailwind.config = {
    darkMode: 'class',
    theme: {
      extend: {
        colors: {
          surface: {
            DEFAULT: '#0f1117',
            raised: '#161b27',
            overlay: '#1e2537',
            border: '#2a3348',
          },
          accent: {
            DEFAULT: '#6366f1',   // indigo-500
            hover:   '#818cf8',   // indigo-400
            muted:   '#312e81',   // indigo-900
          },
          text: {
            primary:   '#e2e8f0',
            secondary: '#94a3b8',
            muted:     '#475569',
          },
        },
        fontFamily: {
          sans: ['Inter', 'ui-sans-serif', 'system-ui', 'sans-serif'],
          mono: ['JetBrains Mono', 'ui-monospace', 'monospace'],
        },
      },
    },
  }
</script>

<!-- Inter + JetBrains Mono from Google Fonts -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
```

---

## Theme

**Always dark.** Apply `class="dark"` on `<html>`. Background is `surface` (`#0f1117`). Never render on a white or light background.

| Token | Hex | Usage |
|---|---|---|
| `surface` | `#0f1117` | Page background |
| `surface-raised` | `#161b27` | Cards, panels |
| `surface-overlay` | `#1e2537` | Popovers, code blocks, nested containers |
| `surface-border` | `#2a3348` | Borders, dividers |
| `accent` | `#6366f1` | Primary highlights, active states, links |
| `accent-hover` | `#818cf8` | Hover states on accent elements |
| `accent-muted` | `#312e81` | Subtle accent backgrounds (badges, tags) |
| `text-primary` | `#e2e8f0` | Body text, headings |
| `text-secondary` | `#94a3b8` | Labels, captions, secondary info |
| `text-muted` | `#475569` | Placeholders, disabled, decorative |

---

## Typography

- **Font:** Inter for all prose; JetBrains Mono for code, identifiers, file paths.
- **Base size:** `text-sm` (14px) for body; `text-xs` for captions and metadata.
- **Line height:** `leading-relaxed` for body paragraphs; `leading-snug` for headings.
- **Headings:** semibold (`font-semibold`), `text-text-primary`. Use `text-lg` → `text-base` → `text-sm` hierarchy. Do not use `text-xl` or larger inside content sections.
- **Code spans:** `font-mono text-accent bg-surface-overlay px-1 rounded`.

---

## Layout

```html
<body class="dark bg-surface text-text-primary font-sans antialiased min-h-screen">
  <div class="max-w-4xl mx-auto px-6 py-10 space-y-10">
    <!-- page title -->
    <header class="border-b border-surface-border pb-6">
      <h1 class="text-lg font-semibold text-text-primary tracking-tight">Feature Name</h1>
      <p class="mt-1 text-sm text-text-secondary">Short subtitle or description</p>
    </header>

    <!-- content sections -->
    <section class="space-y-4">
      <h2 class="text-sm font-semibold uppercase tracking-widest text-accent">Section Title</h2>
      <!-- content -->
    </section>
  </div>
</body>
```

- Max width: `max-w-4xl`, centered.
- Generous vertical rhythm: `space-y-10` between top-level sections.
- Section headings: small-caps style — `text-sm font-semibold uppercase tracking-widest text-accent`.

---

## Cards and panels

```html
<div class="bg-surface-raised border border-surface-border rounded-xl p-5 space-y-3">
  <h3 class="text-sm font-semibold text-text-primary">Card Title</h3>
  <p class="text-sm text-text-secondary leading-relaxed">Content goes here.</p>
</div>
```

- Rounded corners: `rounded-xl`.
- No drop shadows. Separation is achieved through background contrast and a subtle border.

---

## Tables

```html
<div class="overflow-x-auto rounded-xl border border-surface-border">
  <table class="w-full text-sm">
    <thead class="bg-surface-overlay text-text-secondary uppercase text-xs tracking-wider">
      <tr>
        <th class="px-4 py-3 text-left font-medium">Column</th>
      </tr>
    </thead>
    <tbody class="divide-y divide-surface-border">
      <tr class="hover:bg-surface-overlay transition-colors">
        <td class="px-4 py-3 text-text-primary">Value</td>
      </tr>
    </tbody>
  </table>
</div>
```

---

## Diagrams (Mermaid)

Wrap Mermaid diagrams in a centered panel. Configure Mermaid to match the dark theme:

```html
<script>
  mermaid.initialize({
    startOnLoad: true,
    theme: 'base',
    themeVariables: {
      darkMode: true,
      background:      '#161b27',
      primaryColor:    '#312e81',
      primaryBorderColor: '#6366f1',
      primaryTextColor:   '#e2e8f0',
      lineColor:       '#6366f1',
      secondaryColor:  '#1e2537',
      tertiaryColor:   '#1e2537',
      edgeLabelBackground: '#161b27',
      fontFamily: 'Inter, ui-sans-serif, sans-serif',
      fontSize: '13px',
    },
  });
</script>

<div class="bg-surface-raised border border-surface-border rounded-xl p-6 flex justify-center">
  <div class="mermaid text-sm">
    flowchart TD
      A[Start] --> B[Step]
  </div>
</div>
```

---

## Badges and tags

```html
<!-- Neutral -->
<span class="inline-flex items-center px-2 py-0.5 rounded-full text-xs font-medium bg-surface-overlay text-text-secondary border border-surface-border">
  Label
</span>

<!-- Accent -->
<span class="inline-flex items-center px-2 py-0.5 rounded-full text-xs font-medium bg-accent-muted text-accent border border-accent/30">
  Active
</span>
```

---

## Code blocks

```html
<pre class="bg-surface-overlay border border-surface-border rounded-lg p-4 overflow-x-auto text-xs font-mono text-text-primary leading-relaxed"><code>your code here</code></pre>
```

---

## Minimal full-page template

Use this as the starting skeleton for every generated HTML file:

```html
<!DOCTYPE html>
<html lang="en" class="dark">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{Title}}</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          colors: {
            surface: { DEFAULT: '#0f1117', raised: '#161b27', overlay: '#1e2537', border: '#2a3348' },
            accent:  { DEFAULT: '#6366f1', hover: '#818cf8', muted: '#312e81' },
            text:    { primary: '#e2e8f0', secondary: '#94a3b8', muted: '#475569' },
          },
          fontFamily: {
            sans: ['Inter', 'ui-sans-serif', 'system-ui', 'sans-serif'],
            mono: ['JetBrains Mono', 'ui-monospace', 'monospace'],
          },
        },
      },
    }
  </script>
  <!-- Add Mermaid only when the page contains diagrams -->
  <!-- <script src="https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.min.js"></script> -->
  <!-- Add Alpine only when the page needs interactivity -->
  <!-- <script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3/dist/cdn.min.js"></script> -->
</head>
<body class="dark bg-surface text-text-primary font-sans antialiased min-h-screen">
  <div class="max-w-4xl mx-auto px-6 py-10 space-y-10">

    <header class="border-b border-surface-border pb-6">
      <h1 class="text-lg font-semibold text-text-primary tracking-tight">{{Title}}</h1>
      <p class="mt-1 text-sm text-text-secondary">{{Subtitle}}</p>
    </header>

    <!-- Sections go here -->

  </div>
</body>
</html>
```

---

## Tone and aesthetics

- **Elegant and restrained** — no gradients, no glows, no animations unless they carry meaning.
- **High contrast text** — `text-primary` on `surface` always passes WCAG AA.
- **Whitespace is content** — generous padding, never cramped.
- **Accent sparingly** — indigo (`accent`) is used to draw the eye, not to decorate. Reserve it for headings, active states, and key highlights.
- **No decorative icons** unless they aid comprehension (e.g. a checkmark on a completed step).
