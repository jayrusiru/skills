---
name: just-finish-it
description: Use when the user explicitly wants a timeboxed autonomous work session, unattended agent workflow, automatic answers to planning or grill-me questions, or asks an AI assistant to keep working for hours until a goal is complete.
---

# Just Finish It

Run a user-approved autonomous session with a fixed time budget. Treat invocation of this skill as permission to choose safe defaults for routine planning questions, while preserving hard stops for decisions that require the human.

## How To Use

Invoke this skill when you want the assistant to keep moving without pausing for routine planning questions:

```text
Use $just-finish-it for 4 hours to complete this goal: <goal>
```

If your agent platform does not use `$skill-name` syntax, invoke the skill by name and provide the same goal and timebox.

Good examples:

```text
Use $just-finish-it for 8 hours to audit the backlog, fix safe issues, run validation, and leave a handoff.
Use $just-finish-it for 48 hours to finish the current milestone, choosing safe defaults when questions come up.
Use $just-finish-it to complete this refactor unless a hard stop is reached.
```

Include the goal, timebox, allowed scope, and any hard constraints. If no timebox is provided, default to `4 hours`.

## Recommended Companion Skills

Before relying on this skill for long autonomous work, install or make available:

- [`grill-me`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md) for stress-testing the goal and decision tree.
- [`superpowers`](https://github.com/obra/superpowers) for structured brainstorming, planning, execution, and verification workflows.

This skill can still run without those companions by using the fallback behavior below, but it works best when both are available.

## Supporting Skills And Fallbacks

- Use `grill-me`, if available, to stress-test the goal and decision tree. If it is not installed, perform the same question-and-default loop inline.
- Use relevant process skills, if available, for the work: usually brainstorming, planning, execution, and verification. Superpowers skill names may include `superpowers:brainstorming`, `superpowers:writing-plans`, `superpowers:subagent-driven-development`, `superpowers:executing-plans`, and `superpowers:verification-before-completion`.
- If a named supporting skill is unavailable, follow the behavior described here instead of stopping solely because the helper skill is missing.

## Autonomy Contract

Proceed without waiting only when all are true:

1. The user explicitly requested autonomy, unattended work, automatic answers, or a timebox.
2. The choice is local, reversible, no-cost, and does not expose secrets or affect production systems.
3. The selected answer can be defended from the user's prompt, repo instructions, existing code, durable memory, or common engineering practice.

Do not pretend the user answered. Record it as an agent-selected default.

## Non-Interactive Continuation Rule

Invocation of this skill is explicit permission to continue through routine human-review gates in companion workflows. Do not stop for human input unless a Hard Stop applies.

Do not pause at:

- `grill-me` questions.
- Brainstorming design approval gates.
- Spec or plan review gates.
- Writing-plans execution handoffs.
- "Which approach?" prompts.
- "Please review before proceeding" checkpoints.
- "Plan complete" messages that would normally wait for the user.

For each routine gate:

1. Run the required self-review internally.
2. Select the safest default using the Default Answer Rubric.
3. Add a Decision Log entry.
4. Continue immediately into the next planning, execution, or verification step.

Default execution choice: use delegated/subagent execution when the platform supports it and the user explicitly requested autonomous or delegated work; otherwise execute inline. Never end the turn with a question unless a Hard Stop applies.

## Implementation And Source Control Policy

Planning is not a completion state. After brainstorming, spec, or plan work finishes, immediately continue into implementation unless the user's original goal was explicitly planning-only.

Completion means both the requested implementation is done and the user's stated goal is satisfied. If the implementation is done but the goal is not yet met, continue with the next necessary implementation, integration, validation, or cleanup step.

When companion workflows ask to commit specs, plans, checkpoints, or completed tasks, skip the commit and continue. Do not run source-control write actions unless the user's original prompt explicitly requests them.

Do not create commits, check-ins, tags, branches, pull requests, pushes, or source-control submissions by default. Source-control read-only checks such as status, diff, or log are allowed when useful.

## Timebox

Parse the requested budget: `4 hours`, `8h`, `48 hours`, or similar. If omitted, use `4 hours`.

For long budgets, keep a resumable handoff: goal, constraints, decisions, completed work, validation, blockers, and next action. If the platform supports a thread heartbeat or automation and the user asked for multi-hour continuation, create or propose it instead of silently relying on one chat turn to last forever.

Stop when the goal is complete, the timebox expires, or a hard stop is reached.

## Workflow

1. Restate the goal, timebox, and hard-stop policy in one short update.
2. Inspect project instructions and current state before making decisions.
3. Run `grill-me` internally:
   - Generate the next important question.
   - Answer it using the Default Answer Rubric.
   - Add a Decision Log entry.
   - Continue until no high-impact unknowns remain.
4. Use the relevant process flow. When a process skill asks a routine question or requests design/plan approval, apply the same rubric, log the selected default, and continue.
5. Execute with subagents or delegated agents only when the platform supports them and the user explicitly requested subagents, parallel agents, or this autonomous workflow in terms that clearly imply delegated execution. Otherwise execute inline.
6. Do not stop after planning. Continue into implementation and verification until the requested implementation is complete and the user's stated goal is satisfied, the timebox expires, or a Hard Stop is reached.
7. Verify, summarize, and leave a handoff if there is remaining timeboxed work.

## Default Answer Rubric

Choose the answer that best satisfies these priorities, in order:

1. Follow explicit user instructions and repo instructions.
2. Prefer the smallest useful scope that can complete the stated goal.
3. Preserve existing architecture, style, data, source control state, and user changes.
4. Prefer reversible local changes over irreversible or external actions.
5. Prefer proven tools, tests, and existing workflows over new abstractions.
6. Minimize cost, network dependence, credentials use, and operational risk.
7. Choose the option that is easiest to verify within the timebox.

Decision Log format:

```text
Q: <question the workflow would have asked>
Selected default: <answer chosen>
Reason: <evidence and rubric priority>
Confidence: high|medium|low
Revisit if: <condition that would change the answer>
```

If confidence is low and the choice is still safe and reversible, choose the conservative option and continue. If confidence is low and risk is high, stop.

## Hard Stops

Pause and ask the user instead of auto-answering when a decision involves:

- Credentials, secrets, accounts, personal data, email, SMS, payments, or purchases.
- Legal, medical, financial, employment, or compliance judgment.
- Deleting data, destructive source-control actions, production deploys, public releases, or irreversible migrations.
- Changing project ownership, licensing, branding, pricing, or user-facing commitments.
- Installing network dependencies or running commands outside the sandbox when approval is required.
- A test or validation failure that could hide user-facing breakage.
- Two failed attempts at the same blocker or three total failed implementation attempts.

## Progress And Completion

During active work, give concise progress updates at meaningful milestones or about every 30 minutes. At completion, report what changed, validation performed, decisions auto-selected, and any residual risks.
