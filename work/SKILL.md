---
name: work
description: Stepwise implementation workflow for an already confirmed plan. Use when the user invokes $work or asks Codex to start implementing a plan one step at a time, with each step independently implemented, tested, reviewed by a child/subagent when available, fixed until the review passes, and only then followed by the next step. Works especially well after $chat has produced a confirmed step-by-step plan.
---

# Work

## Overview

Use this skill to execute an already confirmed implementation plan in small, verifiable steps. Complete one step at a time, run tests for that step, request an independent child/subagent review when available, fix any issues, repeat review until the step passes, then move to the next step.

## Operating Contract

- Do not use this skill to invent the plan from scratch. If there is no confirmed plan, ask the user to confirm a step-by-step plan first or switch to `$chat`.
- Do not implement multiple planned steps at once unless they are technically inseparable; explain the coupling before proceeding.
- Do not move to the next step while the current step has failing tests, unresolved review findings, or unclear acceptance criteria.
- Do not treat self-review as a substitute for independent review when child/subagent tooling is available.
- Do not make destructive changes, rewrite unrelated areas, or expand scope beyond the confirmed step without user confirmation.
- Preserve user changes in the worktree. Inspect diffs before editing and never revert unrelated work.

## Step Loop

For each plan step, run this loop:

1. Restate the active step.
2. Identify the files, tests, and acceptance criteria for this step.
3. Implement only this step.
4. Run focused verification for this step.
5. Ask a child/subagent to review and test the step when available.
6. If the review fails, fix the issues and repeat verification plus review.
7. Mark the step complete only when implementation, tests, and review all pass.
8. Summarize the completed step and continue to the next step.

## Child Review Protocol

Use a child/subagent for each completed step when the environment supports it. Give the reviewer a bounded prompt:

```text
Review and test only Step N of this implementation.

Plan step:
...

Files changed for this step:
...

Acceptance criteria:
...

Run or inspect the most relevant tests. Report:
- pass/fail
- bugs or regressions
- missing verification
- whether the step is complete enough to proceed
```

The child/subagent should be independent: do not ask it to approve the work by default, and do not hide known failures.

If child/subagent tooling is not available, perform an explicit local review pass yourself and say that independent subagent review was unavailable. The fallback review must still inspect the diff, run relevant tests, and challenge whether the step is complete.

## Review Failure Handling

When review or tests fail:

- Keep the same active step.
- Summarize the failure.
- Fix only issues relevant to the active step.
- Re-run focused verification.
- Re-run child/subagent review when available.
- Continue until the step passes or a real blocker requires user input.

Do not continue to later steps with known failures.

## Verification Expectations

Choose the lightest verification that proves the active step:

- unit tests for pure logic
- integration tests for API, persistence, or cross-module behavior
- e2e/manual browser checks for user-facing workflows
- typecheck/lint when the step touches shared code or typed contracts
- security review when the step touches auth, permissions, secrets, payments, or user-controlled input

Read test output before claiming success. If a verification command cannot run, explain why and use the next-best check.

## Progress Format

Use a compact progress ledger:

```text
Active step: 2 - ...
Status: implementing | verifying | review-failed | complete
Changed files: ...
Verification: ...
Reviewer verdict: ...
Next: ...
```

After each completed step, report:

```text
Step N complete.
Implemented: ...
Verified: ...
Review: passed
Remaining: Step N+1 - ...
```

## Completion

When all steps are complete:

- summarize all changed files
- summarize tests and reviews run
- note any remaining risks or deferred items
- confirm that every planned step passed its verification loop
