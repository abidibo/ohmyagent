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
2 - **Ask clarifying questions**. One at a time, understand purpose, success criteria. Prefer visual representation. If you think it's worth it, create a static html js css frontend for it with diagrams and flows in a tmp dir and xdg-open in the browser to let the user see it.
3 - **Divide the implementation plan in steps**. Ask the user to review the steps, and if he wants to merge them, modify or delete any or add some steps.
4 - **Propose 2-3 approaches**. Each step at a time. Propose 2 or 3 steps with you're recommendation and why. Have the user accept its preference or if he wants something different.
5 - **Implement**. Implement the chosen approach.
6 - **Self review**. Optimize code for DRY principles and fix errors before user delivery
7 - **Review**. Ask the user to review. If the user is satisfied with the implementation, go on, otherwise ask for corrections.
8 - **Repeat**. Repeat from point 4 for the following step.

## Process Flow

This is crucial. This diagram explains exactly what you need to do. Follow it strictly. It's extremely important you follow the rule of implementing step by step.
So you get understanding of the implementation topic, then you ask clarifying question, you divide the implmentation plan in steps, and then you implement each step at a time, looping between step 4 and 6.

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
      exit_condition:
        - The user approves the implementation steps.

    - id: propose_approaches
      title: Propose approaches for current step
      objective: Choose how to execute the current step.
      agent_actions:
        - Focus on one implementation step at a time.
        - Propose 2 or 3 approaches for that step.
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
      to: propose_approaches
      when: user_is_satisfied_and_more_steps_remain

    - from: review
      to: end
      when: user_is_satisfied_and_no_more_steps_remain
```

## Key principles

- **If uncertain on anything, ask!**. If you see potential issues, ask the user to provide more context.
- **One question at a time**. Easier to answer then open ended when possible.
- **DRY implementation**. If you see potential for a function or class or anything to be reused, think about put it ina utils or shared module. **Ask the user for the best place to put this kind of functionality.**
- **Be flexible**. Go back and clarify when something doesn't make sense.

## Mandatory rules

- Do not implement anything until Steps 1-4 have been completed explicitly in the conversation.
- Ask exactly one clarifying question at a time until there is no material ambiguity left.
- After clarification, present the implementation plan as numbered steps and wait for user approval.
- For each step, present 2-3 approaches, state the recommended one, and wait for the user to choose or approve.
- If the user asks to implement immediately, still complete the workflow first unless they explicitly say: "skip the skill workflow".
- If you cannot follow this workflow, say so and stop instead of proceeding.

## Execution gate

Implementation is forbidden until:

 1. requirements are clarified
 2. the step plan is approved
 3. the approach for the current step is approved

!IMPORTANT: The implementation should proceed step by step!
