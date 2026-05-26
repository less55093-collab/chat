---
name: work
description: Stepwise implementation workflow for an already confirmed plan. Use when the user invokes $work or asks Codex to execute a confirmed step-by-step plan with acceptance criteria, focused verification, independent review when available, fixes until review passes, and final whole-change validation. Works especially well after $chat has produced and the user has confirmed an implementation plan. Do not use to invent a plan from scratch.
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

## Startup Gate

Before editing files, confirm that the execution inputs are ready:

1. A step-by-step plan exists and the user has confirmed it, or the user explicitly asked to execute a specific plan.
2. Each step has a purpose, expected work, likely files/modules, acceptance criteria, and focused verification.
3. Dependencies between steps are known enough to choose the first active step.
4. The current worktree status and relevant diffs have been inspected so user-owned changes are preserved.
5. The final whole-change verification commands or manual checks are identified.
6. If the plan references an approved frontend demo, its path/URL and approved design decisions are available and treated as implementation constraints.

If the plan is missing acceptance criteria or verification, derive the smallest reasonable interpretation from the conversation and ask the user to confirm the missing decision before editing. If there is no usable plan, stop and ask the user to confirm a step-by-step plan first or switch to `$chat`.

## Step Loop

For each plan step, run this loop:

1. Restate the active step.
2. Identify the files, tests, acceptance criteria, dependencies, and relevant pre-existing user changes for this step.
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

After all planned steps pass, run a final verification sweep that proves the integrated change still works. Prefer the repository's normal lint, typecheck, unit/integration test, build, or e2e commands when applicable. If the final sweep is too expensive or unavailable, explain the constraint and run the strongest narrower checks that still support the completion claim.

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
- summarize the final whole-change verification result
- for frontend work based on an approved demo, summarize how the implementation matches or intentionally differs from the demo
- note any remaining risks or deferred items
- confirm that every planned step passed its verification loop
