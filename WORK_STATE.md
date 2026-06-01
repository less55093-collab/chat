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
| T-000 | done | Refresh root work state machine for `chat-explore` | none | root skill files | `WORK_STATE.md` | Re-read file and check git status |
| T-001 | done | Create `chat-explore` skill for `/explore` divergent solution generation | T-000 | `chat-explore/**`, skill-creator guidance if needed | `chat-explore/**` | Frontmatter/metadata validation, manual read |
| T-002 | done | Sync `chat-explore` to local installed chat skill | T-001 | `chat-explore/**`, `/Users/th/.codex/skills/chat/**` | `/Users/th/.codex/skills/chat/chat-explore/**` | File comparison and local YAML validation |
| T-003 | done | Whole-change validation and review | T-002 | all changed docs | none unless fixes are needed | Validate skill frontmatter/metadata, scan for scaffold drift, inspect status |

## Active Task

### Complete

Purpose:
Validate the new `chat-explore` project files and local installation.

Child Agent Brief:
Independent verifier reviewed `chat-explore/SKILL.md`, `chat-explore/agents/openai.yaml`, local `/Users/th/.codex/skills/chat/chat-explore/**`, and `WORK_STATE.md`.

Allowed read:
- `chat-explore/**`
- `/Users/th/.codex/skills/chat/**`
- `WORK_STATE.md`

Allowed write:
- None unless fixes are needed after main-agent approval.

Hidden unless requested:
- Any unrelated project implementation plans.

Acceptance Criteria:
- Project `chat-explore` frontmatter and metadata validate.
- Local `chat-explore` files match project files.
- No scaffold placeholders remain.
- Independent verifier review passes.

Verification:
- YAML/frontmatter validation.
- `cmp` project and local files.
- Scaffold scan and diff check.
- Independent verifier review passed after one focused fix.

## File Access Requests

None.

## Transition Log

- 2026-06-01: T-000 pending -> done.
- 2026-06-01: T-001 pending -> ready.
- 2026-06-01: T-001 ready -> assigned -> implementing.
- 2026-06-01: T-001 implementing -> self_check -> main_verify -> needs_fix. Main verification found `chat-explore/agents/openai.yaml` missing the required `interface:` metadata wrapper.
- 2026-06-01: T-001 needs_fix -> assigned -> implementing -> self_check -> main_verify -> done.
- 2026-06-01: T-002 pending -> ready.
- 2026-06-01: T-002 ready -> assigned -> implementing.
- 2026-06-01: T-002 implementing -> self_check -> main_verify -> done.
- 2026-06-01: T-003 pending -> ready.
- 2026-06-01: T-003 ready -> main_verify -> needs_fix. Verifier found missing explicit per-option risk and next-validation fields.
- 2026-06-01: T-003 needs_fix -> main_verify -> done. Added explicit risk and next-validation fields, re-synced local files, and verifier passed.
- 2026-06-01: Work status in_progress -> complete.
