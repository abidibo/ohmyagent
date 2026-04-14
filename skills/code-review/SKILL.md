---
name: code-review
description: Use after completing an implementation. Ask the user if they want a code review, then dispatch one or more reviewer agents in parallel when feasible.
---

# Code Review Skill

Any response that skips a required step is incorrect. Do not optimize for speed over compliance.

## When to use

After completing an implementation, ask the user if they want a code review. If yes, gather context, decide on parallelization, dispatch agents, and act on feedback.

## Workflow

### Step 1 — Ask the user

Ask: **"Do you want me to run a code review on this implementation?"**

If no: skip the entire skill.
If yes: proceed to Step 2.

### Step 2 — Gather context

Before dispatching any agent, collect:

1. **Changed files** — run `git diff --name-only HEAD` (or `git diff --name-only <base>`) to get the list of changed files.
2. **Diff content** — run `git diff HEAD` (or against a base branch) to read what changed.
3. **Plan or spec** — check if a `spec.md` or plan document exists for this work. If it does, note its path. If not, summarize the intent of the changes in 2–4 sentences.

### Step 3 — Decide on parallelization

Analyze the changed files and decide how many reviewers to dispatch:

| Situation | Strategy |
|-----------|----------|
| Single coherent change (one module, one feature, one fix) | 1 reviewer agent |
| Changes span 2+ independent areas (e.g. frontend + backend, API + DB migration) | 1 agent per independent area |
| Large diff with distinct concerns (security, logic, tests) | Split by concern |

**Maximum useful parallelism: 3 agents.** More than that produces overlap without gain.

**Parallelization is feasible when areas are independent** — a reviewer focused on the frontend does not need to read the backend migration to give useful feedback, and vice versa.

### Step 4 — Construct and dispatch reviewer agent(s)

For each reviewer, fill in the `code_reviewer.md` template with:

- **What was implemented**: a concrete description of what this reviewer should evaluate (specific to their area if parallelized)
- **Files to review**: the exact file paths relevant to this reviewer's scope
- **Plan or spec**: either a path to the spec file or an inline summary of requirements/acceptance criteria
- **Focus area** (if parallelized): e.g. "Focus on the Django backend: models, views, serializers, and migrations. Do not review frontend files."

Dispatch all agents in a single message (parallel tool calls).

### Step 5 — Act on feedback

When all agents return:

| Severity | Action |
|----------|--------|
| Critical | Fix immediately before proceeding |
| Important | Fix before closing the work |
| Minor | Note for later; fix if low-effort |

- If the reviewer flags something you disagree with: push back with a technical argument (show the code, cite the spec, explain the reasoning). Do not silently accept wrong feedback.
- If two parallel reviewers contradict each other: flag it to the user with both positions.

## Process Flow

```yaml
workflow:
  name: code_review

  states:
    - id: ask_user
      title: Ask the user
      agent_actions:
        - Ask if the user wants a code review.
      exit_condition:
        - User says yes or no.

    - id: gather_context
      title: Gather context
      agent_actions:
        - Run git diff to identify changed files and their content.
        - Locate any spec or plan document for this work.
        - If no spec exists, write a 2–4 sentence summary of what was implemented and why.
      exit_condition:
        - Changed files known, diff read, plan/spec located or summarized.

    - id: decide_parallelization
      title: Decide on parallelization
      agent_actions:
        - Inspect the changed files for independence.
        - Decide: 1 reviewer or N reviewers (max 3).
        - For N > 1: define each reviewer's scope so they do not overlap.
      exit_condition:
        - Number of reviewers and scope per reviewer decided.

    - id: dispatch
      title: Dispatch reviewer agent(s)
      agent_actions:
        - Fill in the code_reviewer.md template for each agent.
        - Dispatch all agents in a single parallel message.
      exit_condition:
        - All agents return their reports.

    - id: act
      title: Act on feedback
      agent_actions:
        - Fix all Critical issues immediately.
        - Fix all Important issues before proceeding.
        - Note Minor issues.
        - Push back on incorrect feedback with technical reasoning.
      exit_condition:
        - All Critical and Important issues resolved.

  transitions:
    - from: ask_user
      to: end
      when: user_declines

    - from: ask_user
      to: gather_context
      when: user_accepts

    - from: gather_context
      to: decide_parallelization
      when: context_collected

    - from: decide_parallelization
      to: dispatch
      when: scope_defined

    - from: dispatch
      to: act
      when: all_agents_returned

    - from: act
      to: end
      when: all_critical_and_important_issues_resolved
```

## Red Flags

**Never:**
- Dispatch reviewers without reading the diff first — vague prompts produce vague reviews
- Ignore Critical issues
- Proceed past Important issues without fixing them
- Accept reviewer feedback you know is wrong

**If the reviewer is wrong:**
- Push back with technical reasoning
- Show the code or tests that prove it works
- Ask for clarification if the feedback is ambiguous

## Template

See: `code_reviewer.md`
