# Work State

Status: complete
Current task: none
Last updated: 2026-06-01

## Rules

- This file is the source of truth for the active `$work` execution.
- Only `$work` creates or updates this file after the user explicitly invokes `$work`.
- `$chat` may draft this state machine in conversation, but it must not write files.
- Each task must pass main-agent verification before moving to `done`.
- Administrative transitions such as `ready -> assigned -> implementing` require a recorded state update, while `main_verify -> done` and next-task activation require main-agent verification evidence.
- A child agent may directly inspect only the allowed read scope for its task. Access outside that scope requires a file access request to the main agent.

## State Machine

`pending -> ready -> assigned -> implementing -> self_check -> main_verify -> done`

`pending -> blocked`

`main_verify -> needs_fix -> assigned`

`done -> ready(next task)`

## Tasks

| ID | Status | Task | Depends On | Allowed Read | Allowed Write | Verification |
| --- | --- | --- | --- | --- | --- | --- |
| T-000 | done | Create root work state machine | none | root files | `WORK_STATE.md` | Re-read file and check git status |
| T-001 | done | Strengthen `$chat` read-only boundary, understanding checks, and `$work` handoff task capsules | T-000 | `SKILL.md`, `agents/openai.yaml` | `SKILL.md`, `agents/openai.yaml` | Manual read, YAML/frontmatter validation |
| T-002 | done | Redesign `$work` around `WORK_STATE.md`, single-child delegated implementation, file access requests, and main-agent verification | T-001 | `work/SKILL.md`, `work/agents/openai.yaml` | `work/SKILL.md`, `work/agents/openai.yaml` | Manual read, YAML/frontmatter validation |
| T-003 | done | Whole-change validation and review | T-002 | all changed docs | none unless fixes are needed | Validate skill frontmatter/metadata, search for scaffold drift, inspect git diff |

## Active Task

### Complete

Purpose:
Validate the integrated `$chat` and `$work` protocol changes, including the root state machine, metadata, and strict handoff boundary.

Main-agent result:
- Validated all skill frontmatter and agents metadata.
- Searched for scaffold leftovers and old conflicting wording.
- Ran whitespace diff checks.
- Requested independent verifier review.
- Fixed transition-discipline ambiguity found by the verifier.

Acceptance Criteria:
- `$chat` and `$work` agree on strict `$work` handoff.
- `$chat` produces task capsules and drafts `WORK_STATE.md` without writing files.
- `$work` creates/updates `WORK_STATE.md`, delegates one task to one child, handles file access requests, records every state transition, and verifies before `done` or next-task activation.
- Metadata reflects the revised behaviors.

Verification:
- Manual YAML/frontmatter validation passed.
- Agent metadata YAML validation passed.
- Stale wording and scaffold scan passed.
- `git diff --check` passed.
- Independent verifier review passed after one focused fix.

## File Access Requests

None.

## Transition Log

- 2026-06-01: T-000 pending -> done.
- 2026-06-01: T-001 pending -> ready.
- 2026-06-01: T-001 ready -> done.
- 2026-06-01: T-002 pending -> ready.
- 2026-06-01: T-002 ready -> done.
- 2026-06-01: T-003 pending -> ready.
- 2026-06-01: T-003 ready -> done.
- 2026-06-01: Work status in_progress -> complete.
