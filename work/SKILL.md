---
name: work
description: "Execute a confirmed $chat plan after explicit $work invocation. Use $chat's execution-mode decision: simple-main for small tasks handled directly by the main agent without WORK_STATE.md or child agents; state-main for resumable multi-step work handled by the main agent with WORK_STATE.md; delegated-state for larger work that uses WORK_STATE.md plus child-agent implementation and main-agent verification. Do not invent a new plan; consume the confirmed task capsules and verification criteria."
---

# Work

## Overview

Use this skill to execute an already confirmed implementation plan. `$work` does not invent or materially reshape the plan; it consumes `$chat`'s execution-mode decision, task capsules, and verification criteria.

Execution modes:

- `simple-main`: the main agent implements and verifies directly. Do not create `WORK_STATE.md`. Do not spawn a child agent.
- `state-main`: create or update root `WORK_STATE.md`; the main agent implements each task and verifies transitions.
- `delegated-state`: create or update root `WORK_STATE.md`; assign one active task at a time to one child agent, while the main agent owns scope, file-access decisions, diff review, verification, and final completion evidence.

Plan topologies:

- `linear`: execute tasks in order.
- `branched`: execute shared prerequisite tasks first, then try planned branch candidates at the branch point. If a branch fails its defined verification and can be safely rolled back, roll back only that branch's changes and try the next branch.

The shared `$chat -> $work` handoff contract lives in root `TASK_CAPSULE_SCHEMA.md`. `$work` consumes plan and task capsules from that schema; it does not redefine the interface or silently expand the plan.

## Operating Contract

- Run only after the user explicitly invokes `$work`.
- Require a confirmed plan or task capsule from `$chat`, following `TASK_CAPSULE_SCHEMA.md` when available and including execution mode when available.
- If execution mode is missing, make the smallest safe classification before editing: use `simple-main` for small low-risk work, and use `state-main` or `delegated-state` only when the task actually benefits from state or child-agent delegation.
- Do not create `WORK_STATE.md` for `simple-main`.
- Create or update root `WORK_STATE.md` only for `state-main` or `delegated-state`.
- Execute one active task at a time. Do not start the next task until the current task is verified.
- For branched plans, execute only one branch candidate at a time.
- Do not continue past a branch point until one branch passes verification and main-agent review.
- Do not improvise unplanned branches. If all planned branches fail, stop as blocked and report evidence.
- Assign child agents only in `delegated-state`.
- Preserve user changes. Inspect worktree status and relevant diffs before editing or integrating child changes. Never revert unrelated changes.
- Roll back only changes made for the current failed branch. Never use broad destructive commands such as `git reset --hard` or `git checkout -- .` to abandon a branch.
- Do not make destructive changes, rewrite unrelated areas, or expand scope beyond the active task without user confirmation.
- Do not let a child agent approve its own work. The main agent must verify the diff and run or inspect the relevant checks.
- Treat `ready -> assigned -> implementing` as administrative transitions when state is in use. They require a recorded state update but not diff/test verification.
- Treat `self_check -> main_verify -> done` as verification-gated transitions when state is in use. The main agent must inspect the diff and run or review focused checks before marking a task `done` or activating the next task.

## Startup Gate

Before editing implementation files:

1. Confirm the user explicitly invoked `$work`.
2. Confirm a `$chat` plan exists, or that the user supplied an equivalent atomic task with acceptance criteria.
3. Identify execution mode: `simple-main`, `state-main`, or `delegated-state`.
4. Identify plan topology: `linear` or `branched`. If missing, treat the plan as `linear` unless it explicitly describes fallback branches.
5. Confirm the plan and active task contain the required capsule fields from `TASK_CAPSULE_SCHEMA.md`.
6. For a branched plan, confirm each branch has the branch metadata required by `TASK_CAPSULE_SCHEMA.md`, including success criteria, failure criteria, branch-local write scope, rollback boundary, and convergence point.
7. Inspect git status and relevant diffs so user-owned changes are preserved.
8. For `state-main` and `delegated-state`, create or refresh root `WORK_STATE.md` with task IDs, branch topology when relevant, statuses, state rules, active task, file-access log, and transition log.
9. For `simple-main`, skip `WORK_STATE.md` and proceed with a compact main-agent execution loop.
10. Identify the final whole-change verification commands or manual checks.

If critical task boundaries are missing, do not improvise a large implementation. Ask the user to return to `$chat` or provide the missing task details.

## Execution Mode Rules

Choose `simple-main` when all or most are true:

- one small task or a tiny set of tightly coupled edits
- one or two files, or a narrow generated/local sync action
- low-risk and easily reversible
- no meaningful benefit from a child agent
- verification is a simple read, parse, lint, diff, or focused command

Choose `state-main` when:

- several dependent tasks need resumability
- the main agent can implement more safely than a child
- state tracking matters more than delegation

Choose `delegated-state` when:

- tasks have clear ownership and can be handed to a child
- implementation benefits from isolation or independent self-check
- the work is broad enough that state plus delegation reduces risk

## Simple Main-Agent Loop

Use this loop for `simple-main`:

1. Restate the active task and why it is simple enough to skip `WORK_STATE.md`.
2. Inspect relevant files and current diffs.
3. Implement only the active task.
4. Run focused verification.
5. Inspect the diff.
6. Fix any failures locally and re-run verification.
7. Summarize changed files, verification, and any residual risk.

Do not spawn a child agent in this mode.

## WORK_STATE.md Format

Use root `WORK_STATE.md` only for `state-main` or `delegated-state`, unless the user confirmed a different path. Keep it concise and update it at every state transition.

Minimum sections:

- `Status`: `planned`, `in_progress`, `blocked`, or `complete`
- `Current task`
- `Rules`
- `State Machine`
- `Branch Plan`, only when topology is `branched`
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
branch_ready -> branch_attempt -> branch_verify -> branch_done
branch_verify -> branch_failed -> rollback_branch -> branch_ready(next branch)
branch_failed(all branches) -> blocked
```

## Task Capsule Requirements

Each active task must follow the task capsule fields in root `TASK_CAPSULE_SCHEMA.md`.

For `simple-main`, `$work` may infer a missing minor field only when the task is clearly small, low-risk, and the inference is the smallest safe value. Report the inference in the final summary.

For `state-main` and `delegated-state`, missing critical fields block execution until they are supplied. Critical fields include task ID, purpose, exact work, allowed read/write scope, non-goals, acceptance criteria, focused verification, dependencies, and rollback or compatibility notes when relevant.

For `delegated-state`, child-agent context, hidden context, and the file access rule are required before assignment.

For branch points, branch metadata from `TASK_CAPSULE_SCHEMA.md` is required before any branch attempt. If success criteria, failure criteria, branch-local write scope, rollback boundary, or convergence point are missing, stop before editing.

## Branched Plan Loop

Use this loop when plan topology is `branched`.

1. Complete all shared prerequisite tasks before the branch point.
2. At the branch point, record the branch order and branch metadata in `WORK_STATE.md` when state is in use.
3. Before attempting a branch, capture a checkpoint:
   - current git status
   - current diff summary
   - files in the branch's allowed write scope
   - any user-owned or pre-existing changes in those files
4. Attempt only the active branch.
5. Run the branch's focused verification and review loop.
6. If the branch passes, mark the branch selected, record why it passed, and continue from the convergence point.
7. If the branch fails, decide whether rollback is safe:
   - safe: all changes are branch-local and do not overlap user-owned changes
   - unsafe: rollback would touch user-owned changes, migrations, external state, generated assets that cannot be recreated, or files outside the branch rollback boundary
8. If rollback is safe, roll back only the failed branch's own changes and try the next planned branch.
9. If rollback is unsafe, stop and ask the user before reverting anything.
10. If all planned branches fail, stop as blocked with evidence. Do not create a new branch path without user confirmation.

Rollback must be precise. Prefer applying a reverse patch for the branch-local diff or manually undoing only the branch-local edits. Do not use broad repository resets.

## Branch Review Rules

For each branch attempt, the normal verification gate still applies:

- run focused verification for that branch
- use child-agent review in `delegated-state` when available
- inspect the diff before accepting or rolling back
- record failure evidence before moving to the next branch

A branch is considered failed only when it meets the plan's explicit failure criteria or cannot satisfy its success criteria after a reasonable repair loop. Do not abandon a branch just because implementation is inconvenient.

## State-Main Loop

Use this loop for `state-main`:

1. Update `WORK_STATE.md`: task `ready -> implementing`.
2. Implement only the active task.
3. Run focused verification.
4. Update `WORK_STATE.md`: task `implementing -> main_verify`.
5. Inspect the diff and verification output.
6. If verification passes, mark task `done`, update transition log, and activate the next unblocked task.
7. If verification fails, mark `needs_fix`, fix locally, and repeat verification.

## Delegated-State Loop

Use this loop for `delegated-state`:

1. Update `WORK_STATE.md`: task `ready -> assigned`.
2. Build a child-agent brief from the task capsule.
3. Assign the task to one child agent. The child may edit only the allowed write scope and inspect only the allowed read scope.
4. Update `WORK_STATE.md`: task `assigned -> implementing`.
5. While the child works, avoid overlapping edits in the same write scope.
6. When the child reports completion, require a self-check summary: files changed, checks run, risks, and any file access requests made.
7. Update `WORK_STATE.md`: task `implementing -> self_check -> main_verify`.
8. Main agent inspects the diff and runs or reviews focused verification.
9. If verification passes, mark task `done`, update transition log, and activate the next unblocked task.
10. If verification fails, summarize the failure, mark `needs_fix`, and send the same child a repair brief or perform the smallest safe local fix if delegation is unavailable.

Do not continue to later tasks while the current task has failing checks, unresolved diff concerns, or unclear ownership.

## Child Agent Brief

Use this shape only in `delegated-state`:

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

Use this protocol only in `delegated-state`.

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

The main agent must verify every task before it is considered complete.

Use the lightest check that proves the task:

- manual diff review for documentation-only changes
- unit tests for pure logic
- integration tests for APIs, persistence, or cross-module behavior
- e2e/manual browser checks for user-facing workflows
- typecheck/lint when typed or shared contracts changed
- security review when auth, permissions, secrets, payments, or user-controlled input changed

Read verification output before claiming success. If a command cannot run, explain why and run the strongest narrower check available. Record the limitation in `WORK_STATE.md` only when a state file exists.

## Completion

When all tasks are complete:

- run the final whole-change verification sweep identified at startup
- update `WORK_STATE.md` status to `complete` only if a state file exists for this run
- summarize changed files
- summarize execution mode and whether a child agent was used
- summarize plan topology and, for branched plans, which branch was selected or why all branches failed
- summarize final verification result
- note remaining risks, skipped checks, or deferred items
- confirm that every planned task passed verification
