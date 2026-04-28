---
name: spec-implement
description: Implement a spec created by /spec, one step at a time. Each step runs in a fresh subagent (clean context). Tracks progress in progress.yaml. Use when the user wants to implement a spec end-to-end.
---

# Spec-Implement Command

Any response that skips a required step is incorrect. Do not optimize for speed over compliance.

## Purpose

This command implements a spec produced by `/spec`, step by step. Each step is executed by a **fresh subagent** — the main orchestrator (you) manages the plan and state; the subagent gets a clean context and implements exactly one step.

This ensures implementation steps don't bleed context into each other and each subagent focuses entirely on its task.

---

## Workflow

### 0. Locate the spec

1. Ask the user: "Which feature spec would you like to implement?" (unless they already said)
2. Look for `specs/<feature-name>/spec.md`. If not found, tell the user and exit.
3. Look for `specs/<feature-name>/progress.yaml`. If not found, offer to create one from the spec before continuing (invoke `/spec status <feature-name>` style logic to read steps from spec.md and generate the file).

### 1. Load state

Read `specs/<feature-name>/spec.md` and `specs/<feature-name>/progress.yaml`.

Present a status table:

```
Step   Title                          Status        Tasks
─────  ─────────────────────────────  ────────────  ──────────
1      Add JWT middleware              completed     3 / 3
2      Protect routes                 in_progress   1 / 3
3      Write tests                    pending       0 / 3
```

Identify the **resume point**: the first step whose status is not `completed`.
If all steps are completed, inform the user and exit.

### 2. Confirm and start

Ask: "Ready to implement from step N — {title}. Proceed?"
Wait for confirmation before spawning any subagent.

### 3. Step loop

Repeat for each non-completed step, in order:

#### 3a. Announce

Print: `--- Step N / M: {title} ---`

#### 3b. Mark in progress

Update `progress.yaml`: set the step's `status` to `in_progress` and `started_at` to now (ISO 8601). If this is the first step started, also set the top-level `status` to `in_progress` and `started_at`.

#### 3c. Spawn implementation subagent

Spawn a **foreground** subagent via the Agent tool. Pass the following in the prompt — everything the subagent needs, since it starts cold:

```
You are implementing step N of M from a software spec. You have no conversation history — everything you need is in this prompt.

## Spec file: specs/{feature-name}/spec.md

{full content of spec.md}

## Your task: Step N — {step title}

{full content of this step from the spec, including description, sub-tasks, code examples, and file paths}

## Completed so far (summary)

{for each completed step: "Step X — {title}: DONE. Files changed: {list if known}"}
{if none: "No steps completed yet."}

## Instructions

1. Read the relevant files in the codebase before writing anything. Use Glob and Read to understand the current state.
2. Implement this step only — do not touch other steps.
3. Follow the project's existing conventions exactly (naming, structure, patterns).
4. After implementing, report back:
   - A bullet list of every file you created or modified (with the change summary)
   - Any decisions you made that deviate from the spec (and why)
   - Any open questions or issues the user should know about
5. Do not summarize the spec back. Focus only on what you did.
```

#### 3d. Collect result

Read the subagent's report. Print it to the user under a clear heading: `### Step N result`.

#### 3e. Self-review

Spawn a **foreground** self-review subagent. Pass the same context as 3c (spec content, step content, completed-steps summary) plus the implementation subagent's report. Instruct it to:

1. Read every file that was created or modified in this step.
2. Check for bugs, logic errors, missing edge cases, or clear deviations from the spec.
3. Fix any important issues it finds directly in the code.
4. Report back: what it found, what it fixed (with file paths), and what it deliberately left alone.

The self-review subagent must not implement future steps or touch files unrelated to this step. After the self-review subagent completes, print its report under `### Step N self-review`.

#### 3f. Update progress

Update `progress.yaml`:
- Set the step's `status` to `completed`, `completed_at` to now.
- Set all tasks under the step to `completed` with `completed_at`.
- If this was the last step: set top-level `status` to `completed`, `completed_at` to now.
- Write the file.

#### 3g. Next step

Continue the loop with the next non-completed step.

### 4. Completion

When all steps are done:

Print: "All N steps implemented."

Invoke the `code-review` command in post-session mode (scope = all files changed across all steps, parallel agents).

---

## Subagent handoff rules

These rules are non-negotiable:

- **Every step gets a fresh subagent.** Never implement steps in the main conversation — always delegate.
- **Always foreground.** Never spawn background agents — completion must be confirmed before moving on.
- **Full context in the prompt.** The subagent has no memory. Pass spec content, step content, and completed-steps summary explicitly every time.
- **Compact completed-steps summary.** Do not paste full outputs of prior steps — one line per step: what was done, which files changed.
- **One step per subagent.** The subagent implements exactly one step. It must not read ahead or implement future steps.
- **Correction subagents are also fresh.** Do not reuse a subagent for corrections — spawn a new one with the corrected brief.

---

## progress.yaml format

```yaml
feature: <feature-name>
spec: specs/<feature-name>/spec.md
created: <ISO 8601 date>
status: pending          # pending | in_progress | completed
started_at: ~
completed_at: ~

steps:
  - id: 1
    title: "Step title"
    status: pending      # pending | in_progress | completed
    started_at: ~
    completed_at: ~
    tasks:
      - id: "1.1"
        description: "..."
        status: pending
        started_at: ~
        completed_at: ~
```

If `progress.yaml` does not exist, generate it from the spec's Implementation Steps section before starting. Use the same format. Each step gets tasks derived from the step's sub-bullets (not copied from the title).

---

## Key principles

- **Main agent = orchestrator.** You manage state, prompt construction, user interaction, and progress tracking. You do not write code.
- **Subagent = implementor.** It reads files, writes code, and reports back. It has no memory.
- **Self-review is mandatory.** After each implementation subagent, always spawn a self-review subagent to catch and fix bugs before moving on.
- **Progress is durable.** progress.yaml is updated after every step so the session can be resumed across conversations.
- **If you cannot follow this workflow, say so and stop.**
