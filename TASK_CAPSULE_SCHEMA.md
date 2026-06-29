# Task Capsule Schema

This document defines the shared handoff contract between `$chat` and `$work`.

`$chat` produces plan and task capsules after read-only discovery. `$work` consumes those capsules after the user explicitly invokes `$work`. The schema is a behavioral contract for skill output, not a runtime data format or a generated artifact.

## Goals

- Keep `$chat -> $work` handoffs complete, testable, and low-ambiguity.
- Prevent `$chat` and `$work` from drifting into different definitions of a "ready" task.
- Make missing implementation boundaries visible before workspace edits start.
- Give future validation tools a stable set of fields to check.

## Plan Capsule

A plan capsule describes the whole change. `$chat` should include these fields before handing off to `$work`:

| Field | Required | Notes |
| --- | --- | --- |
| Problem statement | Yes | One concise statement of the user-visible problem or desired outcome. |
| Current context | Yes | Evidence from repository inspection, conversation, or both. Separate evidence from inference when useful. |
| Goals | Yes | Outcomes the implementation must achieve. |
| Non-goals | Yes | Explicitly excluded behavior, files, workflows, or quality bars. |
| Deferred decisions | When applicable | Decisions intentionally left outside this work. |
| Reuse decision | Required for tool-like or research-backed work | `direct use`, `wrap/integrate`, `fork/adapt`, `vendor/template import`, `API integration`, `reference-only`, `build custom`, or `not applicable`, with rationale. |
| Adopted source | Required when reuse decision is not `not applicable` | Exact project, package, API, CLI, MCP server, plugin, template, fork, or `none`. Include URL or local path when known. |
| Adoption path | Required when reuse decision is not `not applicable` | How implementation should use the adopted source: install/use directly, wrap, fork, vendor, call API, or use as reference only. |
| Custom-code boundary | Required for tool-like or research-backed work | What local code may be written, and what functionality must remain supplied by the adopted source. |
| Custom Build Justification | Required when reuse decision is `build custom` or when local code duplicates candidate functionality | Evidence that viable candidates failed the core constraints; identify any candidate parts still reusable as references. |
| Execution mode | Yes | `simple-main`, `state-main`, or `delegated-state`, with rationale. |
| Plan topology | Yes | `linear` or `branched`, with rationale. |
| Whole-change acceptance criteria | Yes | User-visible criteria for the full change. |
| Final verification | Yes | Final commands, manual checks, or document review that prove completion. |
| Risks and open questions | Yes | Include "none known" only after checking. |
| Task list | Yes | One or more task capsules, ordered by dependency. |
| Draft `WORK_STATE.md` | Conditional | Include only for `state-main` or `delegated-state`; omit for `simple-main`. |

## Task Capsule

A task capsule describes one independently understandable implementation step.

| Field | Required | Notes |
| --- | --- | --- |
| Task ID and title | Required for stateful work | Use stable IDs such as `T-001` for `state-main` and `delegated-state`; recommended but optional for `simple-main`. |
| Purpose | Yes | Why this task exists. |
| Exact work | Yes | Concrete edits or actions, not vague intent. |
| Reuse source | Required for tool-like or research-backed work | The exact adopted source this task uses, or `none` with justification. |
| Adoption action | Required for tool-like or research-backed work | `install`, `configure`, `wrap`, `fork`, `vendor`, `integrate API`, `test adopted behavior`, `adapt UX`, `reference only`, or `custom build justified`. |
| Custom-code boundary | Required for tool-like or research-backed work | The local code allowed in this task; must not replace adopted-source functionality unless justified. |
| Likely files/modules | Yes | Expected files or directories. Use "unknown, inspect first" only when discovery cannot know. |
| Allowed read scope | Yes | Files, directories, or repository areas the task may inspect. |
| Allowed write scope | Yes | Files, directories, or generated artifacts the task may modify. Use "none" for read-only or verification tasks. |
| Context for child agent | Required for `delegated-state` | Information the child needs to complete only this task. Optional for main-agent modes. |
| Hidden unless requested | Required for `delegated-state` | Context the child should not need unless it requests more access. Optional otherwise. |
| Explicit non-goals | Yes | Prevents scope creep inside the task. |
| Acceptance criteria | Yes | Task-level pass conditions. |
| Focused verification | Yes | The smallest check that proves this task. |
| Dependencies | Yes | Earlier task IDs or `none`. |
| Rollback / compatibility notes | When applicable | Required when edits affect migrations, external state, generated assets, user data, or branch attempts. |
| Branch metadata | Conditional | Required when this task is a branch point. See below. |

## Execution Mode Differences

### `simple-main`

Use for small, low-risk, easily verified work. `$work` must not create `WORK_STATE.md` or spawn child agents.

The capsule may be compact, but it still needs purpose, exact work, acceptance criteria, verification, and scoped files. If a minor field is missing and the task is clearly safe, `$work` may infer the smallest safe value and report that inference.

For tool-like or research-backed `simple-main` work, reuse fields are not minor. If the plan mentions reusable candidates or a reuse decision, `$work` must have enough reuse source, adoption action, and custom-code boundary detail to avoid rebuilding equivalent functionality.

### `state-main`

Use when resumability or multiple dependent steps matter, but delegation is not useful.

Every task should have a stable task ID, status transitions, dependencies, verification, and allowed write scope. `$work` creates or refreshes `WORK_STATE.md` and updates it at each state transition.

### `delegated-state`

Use when tasks are separable enough for child-agent implementation.

Every task must include allowed read scope, allowed write scope, child context, hidden context, non-goals, and the file access request rule. The main agent remains responsible for diff review, verification, and final completion.

## Plan Topology Differences

### `linear`

Use when tasks can be executed in order with no planned fallback branch. If topology is omitted, `$work` may treat the plan as `linear` only when the plan does not describe alternatives or fallback attempts.

### `branched`

Use only when a real implementation uncertainty cannot be resolved through read-only inspection or conversation.

Each branch point must define:

- shared prerequisite tasks before the branch
- branch order, such as `A -> B`
- purpose and exact work for each branch
- files/modules each branch may touch
- branch-local write scope
- success criteria
- failure criteria
- rollback boundary
- convergence point after one branch succeeds
- what to do if all branches fail

`$work` may try the next planned branch only after the current branch fails its defined verification and rollback is safe. If rollback could affect user-owned changes, generated assets, migrations, external state, or files outside the rollback boundary, `$work` must stop and ask the user before reverting.

## Producer Responsibilities: `$chat`

- Inspect enough context before drafting capsules.
- Ask targeted questions until the problem, scope, and acceptance criteria are clear enough.
- Produce capsules that match this schema before handing off to `$work`.
- For tool-like or research-backed work, carry reuse decisions into both the plan capsule and each relevant task capsule.
- If recommending `build custom`, include a concrete Custom Build Justification instead of relying on generic preference or convenience.
- Include a draft `WORK_STATE.md` only for `state-main` or `delegated-state`.
- Keep the handoff read-only. `$chat` must not create this schema's artifacts or implementation files.
- State clearly that confirmation is not implementation permission; only explicit `$work` invocation unlocks edits.

## Consumer Responsibilities: `$work`

- Confirm explicit `$work` invocation before editing.
- Consume the confirmed capsules instead of inventing a new plan.
- For tool-like or research-backed work, verify that task capsules implement the stated reuse decision. If a plan says to reuse but the tasks rebuild equivalent functionality, stop as blocked and ask for a corrected handoff.
- Reject silent custom rebuilds unless the plan includes Custom Build Justification and a clear custom-code boundary.
- Classify missing execution mode only when the task is small and safe enough to do so.
- For broad tasks with missing critical fields, stop as blocked or ask the user to return to `$chat`.
- For branch plans with missing success, failure, rollback, or convergence fields, stop before attempting the branch.
- Preserve user changes and verify each task before moving on.

## Minimal Output Template

```text
Plan Capsule
Problem statement:
Current context:
Goals:
Non-goals:
Deferred decisions:
Execution mode:
Plan topology:
Whole-change acceptance criteria:
Final verification:
Risks and open questions:
Reuse decision:
Adopted source:
Adoption path:
Custom-code boundary:
Custom Build Justification:

Task Capsule
Task ID and title:
Purpose:
Exact work:
Reuse source:
Adoption action:
Custom-code boundary:
Likely files/modules:
Allowed read scope:
Allowed write scope:
Context for child agent:
Hidden unless requested:
Explicit non-goals:
Acceptance criteria:
Focused verification:
Dependencies:
Rollback / compatibility notes:

Branch Metadata, only for branch points
Shared prerequisite tasks:
Branch order:
Branch purpose and exact work:
Branch-local write scope:
Success criteria:
Failure criteria:
Rollback boundary:
Convergence point:
If all branches fail:
```
