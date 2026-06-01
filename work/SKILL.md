---
name: work
description: Execute a confirmed $chat plan through a root WORK_STATE.md state machine, one atomic task at a time. Use when the user explicitly invokes $work after approving a plan with task capsules. The main agent owns state, scope, child-agent assignment, file-access decisions, diff review, focused verification, fix loops, and final validation. A single child agent implements each active task with only the allowed context and must request access to files outside its allowed scope.
---

# Work

## Overview

Use this skill to execute an already confirmed implementation plan. `$work` does not invent or materially reshape the plan; it consumes `$chat` task capsules, creates or refreshes root `WORK_STATE.md`, assigns one atomic task at a time to a single child agent, and verifies the result before moving forward.

The main agent is responsible for orchestration, file-access decisions, integration, tests, state transitions, and final completion evidence. The child agent is responsible for implementing only the assigned task inside its allowed scope.

## Operating Contract

- Run only after the user explicitly invokes `$work`.
- Require a confirmed plan with atomic task capsules. If there is no usable plan, stop and ask the user to return to `$chat`.
- Create or update root `WORK_STATE.md` before implementation begins, using the confirmed task capsules as the source material.
- Treat `WORK_STATE.md` as the source of truth for active task, state transitions, allowed scopes, verification, blockers, and handoff notes.
- Execute one active task at a time. Do not start the next task until the current task is verified and marked `done`.
- Assign each active task to exactly one child agent when child-agent tooling is available.
- Give the child only need-to-know context: task purpose, allowed read/write scopes, acceptance criteria, verification expectations, and explicit non-goals.
- Preserve user changes. Inspect worktree status and relevant diffs before editing or integrating child changes. Never revert unrelated changes.
- Do not make destructive changes, rewrite unrelated areas, or expand scope beyond the active task without user confirmation.
- Do not let a child agent approve its own work. The main agent must verify the diff and run or inspect the relevant checks.
- Treat `ready -> assigned -> implementing` as administrative transitions. They require a recorded state update but not diff/test verification.
- Treat `self_check -> main_verify -> done` as verification-gated transitions. The main agent must inspect the diff and run or review focused checks before marking a task `done` or activating the next task.
- If child-agent tooling is unavailable, record the limitation in `WORK_STATE.md`, perform the smallest safe local fallback, and say that independent delegated implementation was unavailable.

## Startup Gate

Before editing implementation files:

1. Confirm the user explicitly invoked `$work`.
2. Confirm a step-by-step plan exists and was approved after `$chat`, or the user supplied an equivalent atomic task plan.
3. Confirm every task has purpose, work, allowed read scope, allowed write scope, acceptance criteria, verification, dependencies, and rollback or compatibility notes when relevant.
4. Inspect git status and relevant diffs so user-owned changes are preserved.
5. Create or refresh root `WORK_STATE.md` with task IDs, statuses, state machine rules, active task, file-access log, and transition log.
6. Mark only the first unblocked task as `ready`; all dependent tasks remain `pending`.
7. Identify the final whole-change verification commands or manual checks.

If the plan is missing critical task boundaries, do not improvise implementation. Ask the user to return to `$chat` or provide the missing task capsule details.

## WORK_STATE.md Format

Use root `WORK_STATE.md` unless the user confirmed a different path. Keep it concise and update it at every state transition.

Minimum sections:

- `Status`: `planned`, `in_progress`, `blocked`, or `complete`
- `Current task`
- `Rules`
- `State Machine`
- `Tasks`
- `Active Task`
- `File Access Requests`
- `Transition Log`

Default transitions:

```text
pending -> ready -> assigned -> implementing -> self_check -> main_verify -> done
pending -> blocked
main_verify -> needs_fix -> assigned
done -> ready(next task)
```

Task table columns:

- `ID`
- `Status`
- `Task`
- `Depends On`
- `Allowed Read`
- `Allowed Write`
- `Verification`

## Task Capsule Requirements

Each active task must be narrow enough for one child agent to implement without seeing the whole project ambition.

Required fields:

- Task ID and title
- Purpose
- Exact work
- Allowed read scope
- Allowed write scope
- Context to provide to the child agent
- Context to hide unless requested
- Explicit non-goals
- Acceptance criteria
- Focused verification
- Dependencies
- Rollback, migration, or compatibility notes when relevant

If any required field is missing, keep the task in `blocked` or ask the user for clarification instead of assigning vague work.

## Delegated Task Loop

For each active task:

1. Update `WORK_STATE.md`: task `ready -> assigned`.
2. Build a child-agent brief from the task capsule.
3. Assign the task to one child agent. The child may edit only the allowed write scope and may inspect only the allowed read scope.
4. Update `WORK_STATE.md`: task `assigned -> implementing`.
5. While the child works, avoid overlapping edits in the same write scope.
6. When the child reports completion, require a self-check summary: files changed, checks run, risks, and any file access requests made.
7. Update `WORK_STATE.md`: task `implementing -> self_check -> main_verify`.
8. Main agent inspects the diff and runs or reviews focused verification.
9. If verification passes, mark task `done`, update transition log, and activate the next unblocked task.
10. If verification fails, summarize the failure, mark `needs_fix`, and send the same child a repair brief or perform the smallest safe local fix if delegation is unavailable.

Do not continue to later tasks while the current task has failing checks, unresolved diff concerns, or unclear ownership.

## Transition Discipline

Every state transition must be recorded in `WORK_STATE.md`, but not every transition has the same verification burden:

- Administrative transitions (`pending -> ready`, `ready -> assigned`, `assigned -> implementing`) prove scheduling and ownership only. Record the transition and the reason.
- Child completion transitions (`implementing -> self_check`) require the child's completion report.
- Verification transitions (`self_check -> main_verify`, `main_verify -> done`, `main_verify -> needs_fix`) require main-agent review evidence.
- A task may not move to `done`, and the next task may not become active, until main-agent verification passes or the task is explicitly blocked.

## Child Agent Brief

Use this shape when assigning work:

```text
Implement only Task T-XXX.

You are not alone in the codebase. Preserve existing user and agent changes. Do not revert unrelated work.

Purpose:
...

Allowed read:
...

Allowed write:
...

Context provided:
...

Hidden unless requested:
...

Non-goals:
...

Acceptance criteria:
...

Focused verification:
...

File access rule:
You may inspect files inside Allowed read. For anything else, stop and request access using the File Access Request format. Do not read or modify outside the allowed scope without approval.

Report:
- files changed
- checks run and results
- risks or blockers
- file access requests
```

## File Access Requests

A child agent must request access before reading or modifying files outside the allowed scope.

Request format:

```text
File Access Request:
- Requested file/path:
- Why needed:
- What decision depends on it:
- Minimum useful scope: summary | excerpt | full file | expanded write scope
- Risk if not granted:
```

Main-agent decision options:

- `deny`: file is unrelated or would expand scope.
- `summarize`: main agent reads the file and gives a short summary.
- `excerpt`: main agent provides only the relevant function, type, config, or test snippet.
- `grant-read`: child may inspect the full file.
- `grant-write`: child may edit the file; update `WORK_STATE.md` allowed write scope first.

Prefer the smallest disclosure that lets the child finish the active task. Record every request and decision in `WORK_STATE.md`.

## Main Verification

The main agent must verify every task before it is marked `done`.

Use the lightest check that proves the task:

- manual diff review for documentation-only changes
- unit tests for pure logic
- integration tests for APIs, persistence, or cross-module behavior
- e2e/manual browser checks for user-facing workflows
- typecheck/lint when typed or shared contracts changed
- security review when auth, permissions, secrets, payments, or user-controlled input changed

Read verification output before claiming success. If a command cannot run, explain why, record the limitation in `WORK_STATE.md`, and run the strongest narrower check available.

## Repair Loop

When checks or review fail:

- Keep the same active task.
- Mark task `needs_fix` in `WORK_STATE.md`.
- Summarize the failure with concrete evidence.
- Send the same child a bounded repair brief when available.
- Keep the same allowed write scope unless the failure proves a scope change is required.
- Re-run focused verification after the fix.
- Repeat until the task passes or a real blocker requires user input.

## Progress Format

Use a compact progress ledger:

```text
Active task: T-002 - ...
Status: assigned | implementing | self_check | main_verify | needs_fix | done
Child agent: ...
Allowed read: ...
Allowed write: ...
Changed files: ...
Verification: ...
State update: ...
Next: ...
```

After each completed task:

```text
Task T-002 complete.
Implemented: ...
Verified: ...
State: done
Remaining: T-003 - ...
```

## Completion

When all tasks are complete:

- run the final whole-change verification sweep identified at startup
- update `WORK_STATE.md` status to `complete`
- summarize changed files
- summarize child-agent work and main-agent verification
- summarize final verification result
- note remaining risks, skipped checks, or deferred items
- confirm that every planned task reached `done`
