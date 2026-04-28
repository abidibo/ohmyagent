---
name: implement
description: Trigger when the user just needs to implement something specific, not a full feature. Use especially when invoked directly.
---

# Implement skill

Any response that skips a required step is incorrect. Do not optimize for speed over compliance.

## When to use

Use when the user wants to implement some tasks, not full features. Especially when invoked directly.

## Workflow

You MUST follow strictly this workflow:

1 - **Explore the implementation details**. Ask the user to provide all needed details for the implementation, until you think the plan is clear enough to proceed. Read files you think are relevant, git logs, commits and anything you think can help you understand better the context.
2 - **Ask clarifying questions**. One at a time, understand purpose, success criteria. Important! Ask all the questions you need to properly understand the context of the implementation. Prefer visual representation. If you think it's worth it, create a static html js css frontend for it with diagrams and flows in a tmp dir and xdg-open in the browser to let the user see it.
3 - **Divide the implementation plan in steps**. Ask the user to review the steps, and if he wants to merge them, modify or delete any or add some steps.
4 - **Propose 2-3 approaches**. Each step at a time. Print brief summary of the step description. Propose 2 or 3 different approaches with your recommendation and why. Choose different approaches, think also as a human, so try to provide differ manners of doing the same functionality. Have the user accept its preference or if he wants something different.
5 - **Implement**. Implement the chosen approach.
6 - **Self review**. Optimize code for DRY principles and fix errors before user delivery
7 - **Review**. Ask the user to review. If the user is satisfied with the implementation, go on, otherwise ask for corrections.
8 - **Code review gate (optional)**. Ask the user if they want to run a code review on this step. If yes, invoke the code-review skill. If no, skip. After the final step, offer a full-session code review instead.
9 - **Repeat**. Repeat from point 4 for the following step.

## Process Flow

This is crucial. This diagram explains exactly what you need to do. Follow it strictly. It's extremely important you follow the rule of implementing step by step.
So you get understanding of the implementation topic, then you ask clarifying question, you divide the implmentation plan in steps, and then you implement each step at a time, looping between step 4 and 8. After each step review, offer the optional code review gate. After the final step, offer a full-session code review.

```yaml
workflow:
  name: strict_iterative_implementation
  description: >
    The agent must follow this workflow strictly for any implementation task.
    It must not skip steps or combine them unless the user explicitly requests changes.

  global_rules:
    - Ask all the clarifying question you need, one at a time during clarification.
    - Prefer visual representations when helpful.
    - Do not implement before the user has reviewed the implementation steps.
    - For each implementation step, propose 2 to 3 approaches.
    - Always include a recommendation and rationale when proposing approaches.
    - After implementation, ask the user to review the result.
    - If the user is not satisfied, collect corrections and revise before moving on.
    - After a step is accepted, continue with the next step starting from approach selection.

  states:
    - id: explore_details
      title: Explore implementation details
      objective: Gather all details needed to understand the implementation.
      agent_actions:
        - Ask the user for missing implementation details.
        - Continue gathering details until the plan is clear enough to proceed.
        - Check if the user is implementing from a spec: look for a spec.md file in any specs/ subdirectory relevant to this task.
        - If a spec folder is found, check for the presence of a progress.yaml file in that same folder.
        - If progress.yaml exists:
            - Load it and use it throughout the session to track task completion.
            - Identify the resume point: find the first step whose status is not "completed".
            - If all steps are completed, inform the user: "progress.yaml shows all steps completed. Nothing left to implement."
              Then exit — do not enter the implementation workflow.
            - If some steps are completed and some are not, inform the user:
              "Found progress.yaml — resuming from step <N> (<step title>). Steps 1–<N-1> are already completed and will be skipped."
            - If no steps are completed yet, inform the user:
              "Found progress.yaml — I'll update it as each step is completed."
            - Skip all steps whose status is "completed" when entering the propose_approaches loop.
        - If progress.yaml does not exist, do not mention it or create it.
      exit_condition:
        - The plan is clear enough to proceed.

    - id: clarify
      title: Ask clarifying questions
      objective: Understand purpose and success criteria.
      agent_actions:
        - Ask one clarifying question at a time.
        - Clarify purpose.
        - Clarify success criteria.
        - Prefer asking for or offering a visual representation when useful.
      exit_condition:
        - Purpose and success criteria are understood.

    - id: divide_steps
      title: Divide implementation into steps
      objective: Build a step-by-step implementation plan, display them in a table if possible.
      agent_actions:
        - Break the implementation into discrete steps.
        - Present the steps to the user for review.
        - Ask whether the user wants to merge, modify, delete, or add steps.
        - Once steps are approved, calculate the total potential agent usage for this session:
            post_step_agents = number_of_steps × 5
            final_review_agents = 6
            total = post_step_agents + final_review_agents
        - Warn the user if total > 20: "Note — this session could use up to {total} agents if you accept every post-step code review. If your runtime has a session agent limit, consider skipping per-step reviews and only running the final full review."
      exit_condition:
        - The user approves the implementation steps.

    - id: propose_approaches
      title: Propose approaches for current step
      objective: After a brief description of the step, choose how to execute the current step.
      agent_actions:
        - Focus on one implementation step at a time.
        - Propose 2 or 3 approaches for that step.
        - Prefer a visual representation to explain the approaches.
        - Recommend one approach.
        - Explain why the recommendation is preferred.
        - Ask the user to choose or suggest a different approach.
      exit_condition:
        - The user selects or approves an approach for the current step.

    - id: implement
      title: Implement chosen approach
      objective: Execute the selected approach for the current step.
      agent_actions:
        - Implement the chosen approach only for the current step.
      exit_condition:
        - Implementation for the current step is complete.

    - id: self_review
      title: Automated Refactoring & Quality Check
      objective: Optimize code for DRY principles and fix errors before user delivery.
      agent_actions:
        - Analyze the implementation for redundancy, DRY violations, or bugs.
        - If issues are found: rewrite the code to apply fixes.
        - If no issues remain: proceed to user review.
      exit_condition:
        - Implementation meets DRY standards and is bug-free.

    - id: review
      title: Review implementation
      objective: Validate the result with the user.
      agent_actions:
        - Ask the user to review the implementation.
        - If the user is not satisfied, ask for corrections.
        - Apply corrections and re-review until satisfied.
      exit_condition:
        - The user is satisfied with the current step.

    - id: update_progress
      title: Update progress file
      objective: Mark the current step and its tasks as completed in progress.yaml, with timestamps.
      agent_actions:
        - If progress.yaml was found at session start:
            - Capture the current ISO 8601 datetime (e.g. 2026-04-15T14:32:00).
            - Set the current step's status to "completed" and set its completed_at to now.
              If the step has no started_at yet, also set started_at to now.
            - Set all tasks under the current step to status "completed" and set their completed_at to now.
              If a task has no started_at, set it to now as well.
            - If this is the first step being completed, set the top-level status to "in_progress"
              and set the top-level started_at to now (if not already set).
            - If this is the last step, set the top-level status to "completed"
              and set the top-level completed_at to now.
            - Write the updated progress.yaml back to disk.
        - If no progress.yaml exists, skip silently.
      exit_condition:
        - progress.yaml updated, or skipped if not present.

    - id: code_review_gate
      title: Offer code review (optional)
      objective: Ask the user if they want a code review on the current step or full session.
      agent_actions:
        - After the user approves a step, ask "Would you like to run a code review on this step before moving on?"
        - If the user declines, skip and continue to the next step or end.
        - If the user accepts, invoke the code-review skill in post-step mode (lighter review, 5 agents, scope = current step's changed files).
        - After the final step is approved, ask "Would you like to run a full code review on the entire implementation?"
        - If the user accepts, invoke the code-review skill in post-session mode (full review, 6 agents, scope = all changed files).
        - If the user declines, skip and end.
      exit_condition:
        - The user declines the review, OR the code review skill completes.

  transitions:
    - from: explore_details
      to: clarify
      when: plan_is_clear

    - from: clarify
      to: clarify
      when: needs_more_clarification

    - from: clarify
      to: divide_steps
      when: purpose_and_success_criteria_are_understood

    - from: divide_steps
      to: divide_steps
      when: user_requests_step_changes

    - from: divide_steps
      to: propose_approaches
      when: user_approves_steps

    - from: propose_approaches
      to: propose_approaches
      when: user_requests_different_options

    - from: propose_approaches
      to: implement
      when: user_selects_approach

    - from: implement
      to: self_review
      when: current_step_is_implemented

    - from: self_review
      to: self_review
      when: issues_found_and_fixed

    - from: self_review
      to: review
      when: self_review_passed_no_issues_remaining

    - from: review
      to: implement
      when: user_requests_corrections

    - from: review
      to: update_progress
      when: user_is_satisfied

    - from: update_progress
      to: code_review_gate
      when: progress_updated_or_skipped

    - from: code_review_gate
      to: propose_approaches
      when: code_review_done_or_skipped_and_more_steps_remain

    - from: code_review_gate
      to: end
      when: code_review_done_or_skipped_and_no_more_steps_remain
```

## Key principles

- **If uncertain on anything, ask!**. If you see potential issues, ask the user to provide more context.
- **One question at a time**. Easier to answer then open ended when possible.
- **DRY implementation**. If you see potential for a function or class or anything to be reused, think about put it ina utils or shared module. **Ask the user for the best place to put this kind of functionality.**
- **Be flexible**. Go back and clarify when something doesn't make sense.

## Mandatory rules

- Do not implement anything until Steps 1-4 have been completed explicitly in the conversation.
- Add brief comments to each function/class targeting an experienced developer.
- Ask all the clarifying questions you need to understand the context, one at a time until there is no material ambiguity left.
- After clarification, present the implementation plan as numbered steps and wait for user approval.
- For each step, present 2-3 approaches (prefer a visual representation), state the recommended one, and wait for the user to choose or approve.
- If the user asks to implement immediately, still complete the workflow first unless they explicitly say: "skip the skill workflow".
- If you cannot follow this workflow, say so and stop instead of proceeding.

## Execution gate

Implementation is forbidden until:

 1. requirements are clarified
 2. the step plan is approved
 3. the approach for the current step is approved

!IMPORTANT: The implementation should proceed step by step!
!IMPORTANT: The developer should always stay in the loop and uderstand well the context of the implementation!
!IMPORTANT: The single step implementation should be easy to follow and understand!
