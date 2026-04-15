---
name: code-review
description: Run a structured, multi-agent code review. Use after the implement skill (per-step or full session) or standalone when the user asks for a code review.
---

# Code Review Skill

Any response that skips a required step is incorrect. Do not optimize for speed over compliance.

## When to use

- **After the implement skill**: at the end of each step (lighter review) or after the full implementation session (comprehensive review). Always ask the user first — if they decline, skip entirely.
- **Standalone**: when the user explicitly asks for a code review or invokes `/code-review`.

## Modes

### 1. Post-step review (during implement workflow)

- Triggered: after each implement step's review phase, before moving to the next step.
- **You MUST ask the user**: "Would you like to run a code review on this step?" If the user says no, skip immediately and continue the implement workflow.
- Scope: only the files changed in the current step.
- Lighter review: run 5 agents **sequentially** (not in parallel) to conserve the session's agent budget. Skip git history check.

### 2. Post-session review (during implement workflow)

- Triggered: after all implement steps are completed.
- **You MUST ask the user**: "Would you like to run a full code review on the entire implementation?" If the user says no, skip entirely.
- Scope: all files changed during the entire implementation session.
- Full review: run all 6 parallel agents.

### 3. Standalone review

- Triggered: user invokes `/code-review` or asks for a code review outside of the implement workflow.
- Scope determination: ask the user how to determine the scope. Options include:
  - Diff against a branch (e.g. main)
  - Specific files or directories
  - Staged/unstaged changes
- Spec determination: ask the user if they have a spec or requirements document to check against. If yes, use it for spec compliance. If no, skip the spec compliance agent.
- Full review: run all applicable agents (5 or 6 depending on spec availability).

## Workflow

```yaml
workflow:
  name: parallel_code_review
  description: >
    Spawn parallel review agents, collect findings, present results in a
    categorized table, auto-fix critical/important issues, and handle minor issues.

  global_rules:
    - This skill is ALWAYS conditional. Never run it without asking the user first.
    - If the user declines, skip immediately with no further discussion.
    - Post-step mode: run agents sequentially to conserve session agent budget.
    - Post-session and standalone mode: run agents in parallel for speed.
    - Always run agents in foreground mode (never background) to guarantee completion before dismissal.
    - Before spawning a new batch, verify no agents from a previous review in this session are still open. If any are, close them first.
    - After all agents in a batch complete, explicitly dismiss or close each one before proceeding.
    - Findings are categorized as critical, important, or minor.
    - Critical and important issues are auto-fixed without individual confirmation.
    - Minor issues require user decision.

  states:
    - id: gate
      title: Ask for permission
      objective: Determine if the user wants to run the review.
      agent_actions:
        - Ask the user if they want to run a code review.
        - If declined, skip entirely and return control.
      exit_condition:
        - User accepts or declines.

    - id: determine_scope
      title: Determine review scope
      objective: Identify which files and context to review.
      agent_actions:
        - In post-step mode, scope is the current step's changed files (automatic).
        - In post-session mode, scope is all files changed during the session (automatic).
        - In standalone mode, ask the user to specify scope.
        - In standalone mode, ask the user if they have a spec to check against.
      exit_condition:
        - Scope and spec availability are determined.

    - id: spawn_agents
      title: Spawn review agents
      objective: Run all applicable review agents and collect findings.
      agent_actions:
        - In post-step mode, spawn agents one at a time (sequentially). Wait for each to complete and dismiss it before spawning the next.
        - In post-session and standalone mode, spawn all agents in parallel. Wait for all to complete, then dismiss each one.
        - Each agent receives only the files in scope (not the entire codebase) and relevant context.
        - Before spawning, verify no agents from a previous review in this session are still open. Close any that are.
        - After all agents complete, explicitly dismiss or close each agent to free session resources.
      exit_condition:
        - All agents have returned their findings and have been dismissed.

    - id: present_findings
      title: Present findings
      objective: Show all issues in a categorized table.
      agent_actions:
        - Merge all agent results.
        - Present a single table with columns - Severity | Category | File | Line(s) | Description.
        - Sort by severity (critical first, then important, then minor).
        - If no issues found, report clean review and exit.
      exit_condition:
        - Findings are presented to the user.

    - id: fix_critical_important
      title: Auto-fix critical and important issues
      objective: Fix all critical and important issues automatically.
      agent_actions:
        - Fix all critical and important issues at once.
        - Show the user a summary of all changes made.
        - Run a quick self-review to ensure fixes don't introduce new issues.
      exit_condition:
        - All critical and important issues are resolved.

    - id: handle_minor
      title: Handle minor issues
      objective: Let the user decide what to do with minor issues.
      agent_actions:
        - Ask the user if they want to address minor issues now or save them for later.
        - If address now, fix them all and show changes.
        - If save for later, ask the user where to save the report file.
        - Write the minor issues report to the user-specified location.
      exit_condition:
        - Minor issues are either fixed or saved.

  transitions:
    - from: gate
      to: end
      when: user_declines

    - from: gate
      to: determine_scope
      when: user_accepts

    - from: determine_scope
      to: spawn_agents
      when: scope_determined

    - from: spawn_agents
      to: present_findings
      when: all_agents_completed

    - from: present_findings
      to: end
      when: no_issues_found

    - from: present_findings
      to: fix_critical_important
      when: critical_or_important_issues_exist

    - from: present_findings
      to: handle_minor
      when: only_minor_issues_exist

    - from: fix_critical_important
      to: handle_minor
      when: minor_issues_exist

    - from: fix_critical_important
      to: end
      when: no_minor_issues

    - from: handle_minor
      to: end
      when: minor_issues_handled
```

## Review Agents

Agents are spawned via the Agent tool. Mode determines parallelism (see global rules). Every agent receives:
- Only the files in scope (changed files for the current step or session — not the full codebase)
- The relevant file contents for context
- The spec/requirements (if available, for spec compliance)

Each agent must be explicitly dismissed after its results are collected.

### Agent 1: Spec Compliance

- **Skip if**: no spec is available (standalone mode without spec).
- **Goal**: verify that the implementation matches the spec/requirements.
- **Checks**:
  - All required features are implemented.
  - No extra unrequested behavior was added.
  - Edge cases mentioned in the spec are handled.
- **Output**: list of findings with severity, file, line(s), and description.

### Agent 2: Code Quality

- **Goal**: review code for quality and maintainability.
- **Checks**:
  - Naming conventions (variables, functions, classes).
  - Function/method length and complexity.
  - Error handling correctness.
  - Type safety and proper use of language features.
  - Readability and clarity.
- **Output**: list of findings with severity, file, line(s), and description.

### Agent 3: Architecture Adherence

- **Goal**: verify the implementation follows existing project patterns and conventions.
- **Checks**:
  - Compare with similar modules/components in the codebase.
  - Correct use of existing abstractions and utilities.
  - Proper module boundaries and separation of concerns.
  - Consistent patterns with the rest of the project.
- **Output**: list of findings with severity, file, line(s), and description.

### Agent 4: DRY Principle

- **Goal**: identify code duplication and reuse opportunities.
- **Checks**:
  - Duplicated logic across the changed files.
  - Duplicated logic between changed files and existing codebase.
  - Opportunities to extract shared utilities or helpers.
  - Repeated patterns that could be abstracted.
- **Output**: list of findings with severity, file, line(s), and description.

### Agent 5: Bug Detection

- **Goal**: find potential bugs, race conditions, and logic errors.
- **Checks**:
  - Off-by-one errors, null/undefined handling.
  - Race conditions or concurrency issues.
  - Resource leaks (unclosed connections, missing cleanup).
  - Incorrect boolean logic or control flow.
  - Security vulnerabilities (injection, XSS, etc.).
- **Output**: list of findings with severity, file, line(s), and description.

### Agent 6: Git History Check

- **Only in**: post-session and standalone modes (skipped in post-step mode).
- **Goal**: review the git history for quality.
- **Checks**:
  - Commits are well-structured (not too large, logical grouping).
  - No accidental files committed (secrets, build artifacts, .env files).
  - Commit messages are meaningful and descriptive.
- **Output**: list of findings with severity, file, line(s), and description.

## Output Format

All findings are presented in a single markdown table:

```
| Severity | Category | File | Line(s) | Description |
|----------|----------|------|---------|-------------|
| CRITICAL | Bug Detection | src/auth.py | 42-45 | SQL injection via unsanitized user input |
| IMPORTANT | DRY | src/utils.py, src/helpers.py | 10, 25 | Duplicated validation logic |
| MINOR | Code Quality | src/models.py | 88 | Variable name `x` is not descriptive |
```

## Integration with Implement Skill

The implement skill's review state (step 7) should include the following after the user approves the step:

> "Would you like to run a code review on this step before moving on?"

And after the final step is completed:

> "Would you like to run a full code review on the entire implementation?"

These are **opt-in only**. If the user says no, proceed without any review.

## Key Principles

- **Always conditional**. Never run without explicit user consent.
- **Sequential in post-step, parallel in post-session/standalone**. Conserve session agent budget across repeated reviews.
- **Foreground agents only**. Never spawn background agents — foreground guarantees completion before dismissal.
- **Always dismiss agents**. Treat agents like open connections: close them explicitly after results are collected.
- **Pre-spawn checkpoint**. Before launching a new batch, confirm no agents from a prior review are still open.
- **Scope containment**. Pass only the relevant changed files to each agent, never the full codebase.
- **Actionable output**. Every finding must include file, line(s), and a clear description.
- **Auto-fix critical/important**. Don't waste the user's time on obvious fixes.
- **User decides on minor**. Respect the user's time and priorities.
