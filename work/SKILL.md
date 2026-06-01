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

## Operating Contract

- Run only after the user explicitly invokes `$work`.
- Require a confirmed plan or task capsule from `$chat`, including execution mode when available.
- If execution mode is missing, make the smallest safe classification before editing: use `simple-main` for small low-risk work, and use `state-main` or `delegated-state` only when the task actually benefits from state or child-agent delegation.
- Do not create `WORK_STATE.md` for `simple-main`.
- Create or update root `WORK_STATE.md` only for `state-main` or `delegated-state`.
- Execute one active task at a time. Do not start the next task until the current task is verified.
- Assign child agents only in `delegated-state`.
- Preserve user changes. Inspect worktree status and relevant diffs before editing or integrating child changes. Never revert unrelated changes.
- Do not make destructive changes, rewrite unrelated areas, or expand scope beyond the active task without user confirmation.
- Do not let a child agent approve its own work. The main agent must verify the diff and run or inspect the relevant checks.
- Treat `ready -> assigned -> implementing` as administrative transitions when state is in use. They require a recorded state update but not diff/test verification.
- Treat `self_check -> main_verify -> done` as verification-gated transitions when state is in use. The main agent must inspect the diff and run or review focused checks before marking a task `done` or activating the next task.

## Startup Gate

Before editing implementation files:

1. Confirm the user explicitly invoked `$work`.
2. Confirm a `$chat` plan exists, or that the user supplied an equivalent atomic task with acceptance criteria.
3. Identify execution mode: `simple-main`, `state-main`, or `delegated-state`.
4. Confirm the active task has purpose, work, likely files/modules, acceptance criteria, focused verification, and dependencies when relevant.
5. Inspect git status and relevant diffs so user-owned changes are preserved.
6. For `state-main` and `delegated-state`, create or refresh root `WORK_STATE.md` with task IDs, statuses, state rules, active task, file-access log, and transition log.
7. For `simple-main`, skip `WORK_STATE.md` and proceed with a compact main-agent execution loop.
8. Identify the final whole-change verification commands or manual checks.

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

## Task Capsule Requirements

Each active task should include:

- Task ID and title when state is in use
- Purpose
- Exact work
- Likely files/modules
- Allowed read scope when delegation is used
- Allowed write scope when delegation is used
- Context to provide to the child agent when delegation is used
- Context to hide unless requested when delegation is used
- Explicit non-goals
- Acceptance criteria
- Focused verification
- Dependencies
- Rollback, migration, or compatibility notes when relevant

If required fields are missing for a broad task, keep the task blocked or ask for clarification instead of assigning vague work.

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
- summarize final verification result
- note remaining risks, skipped checks, or deferred items
- confirm that every planned task passed verification
