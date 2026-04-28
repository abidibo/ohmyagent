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
