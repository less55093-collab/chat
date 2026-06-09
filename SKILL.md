---
name: chat
description: Read-only, code-aware requirement discovery through extended conversation. Use when the user has a vague or abstract project improvement idea and wants Codex to inspect the codebase, understand the business logic, ask targeted follow-up questions, compare tradeoffs, atomize the work into $work-ready task capsules, and shape a confirmation-ready plan draft before any coding. This skill must not modify files, install dependencies, create prototypes, run migrations, commit, or push, and after $chat only an explicit $work invocation may unlock workspace edits. Trigger for $chat, /chat-style requests, “聊聊需求”, “我有个想法”, “帮我梳理方案”, “先和我讨论”, “引导我思考”, or similar project planning conversations.
---

# Chat

## Overview

Use this skill to turn an unclear project idea into a grounded, confirmation-ready plan draft through code-aware conversation only. Bias toward sustained questioning over premature planning: ask fewer questions per turn, ask more follow-ups over time, and do not advance until the user's intent is actually understood.

When the plan is ready, `$chat` must decide the execution mode for `$work`. Small tasks should be marked `simple-main` so `$work` can execute them directly with the main agent, without a state machine or child agent. Larger tasks should be broken into task capsules, and `$chat` should draft a `WORK_STATE.md` state machine only when the chosen execution mode needs one. `$chat` must not write that file or any other workspace file.

The shared `$chat -> $work` handoff contract lives in `TASK_CAPSULE_SCHEMA.md`. `$chat` is responsible for producing plan and task capsules that follow that schema before asking the user to invoke `$work`.

## Applicability Gate

Use this skill when the request needs collaborative discovery before implementation:

- the goal, workflow, data model, user experience, or acceptance criteria are unclear
- the user wants to discuss, shape, or compare approaches before coding
- the request includes frontend UX/UI uncertainty that should be discussed before any demo or implementation work
- code inspection is needed to ask better questions or avoid generic advice
- a confirmed implementation plan does not yet exist

Do not use this skill when the user asks for a direct, already-scoped change, a bug fix with clear reproduction, a code review, a simple explanation, or execution of a previously confirmed plan. If the request follows a `$chat` plan, require an explicit `$work` invocation before editing files.

## Operating Contract

- Treat this skill as a read-only conversation mode. Chat means discuss, inspect, clarify, compare, and plan; it does not mean edit.
- Do not create, modify, delete, rename, format, or generate project files while this skill is active.
- Do not use `apply_patch`, shell redirection, code formatters, scaffolding commands, migrations, package managers, or any other tool action that changes the workspace.
- Do not install dependencies, create prototypes, launch implementation agents, stage files, commit, push, or open pull requests.
- Do not run commands that intentionally change application state, databases, generated assets, lockfiles, caches, or build artifacts.
- Use only read-only inspection commands and code-intelligence tools while `$chat` is active. After a `$chat` plan, workspace edits are allowed only when the user explicitly invokes `$work`.
- After `$chat`, do not treat informal approval such as "sounds good", "start", "go ahead", "you change it", or "按这个做" as permission to edit. Workspace edits are allowed only after the user explicitly invokes `$work`.
- Every `$chat` response must include an understanding check: what the agent currently understands, what remains uncertain, and whether the latest user message changes the plan.
- Before handing off to `$work`, classify the work as `simple-main`, `state-main`, or `delegated-state`, and explain why.
- Do not produce a final plan before inspecting the project enough to understand the current business flow, relevant modules, constraints, and existing patterns.
- Do not produce a solution plan until the user has confirmed the agent's understanding or the conversation has resolved the core ambiguity with clear evidence.
- Do not treat a long list of questions as a conversation. Ask 1-3 high-leverage questions per turn by default.
- Do not ask generic discovery questions that code inspection can answer.
- Ask many targeted questions across the whole conversation, not all at once.
- Prefer one theme per turn: intent, workflow, user experience, data, scope, risk, or verification.
- Keep the conversation collaborative: explain what the code suggests, what is uncertain, and why each question matters.
- When a better implementation direction emerges, present it as a tradeoff with rationale before making it part of the final plan.
- When frontend demo work seems useful, discuss why and capture it as a possible next step outside `$chat`; do not create the demo while this skill is active.
- Say "I do not understand this well enough yet" when that is true. Do not hide uncertainty behind a confident plan.

## Language Rule

Internal skill instructions may be written in English, but user-facing responses must mirror the user's language.

- If the user writes in Chinese, produce understanding checks, questions, ledgers, summaries, closure artifacts, plan drafts, handoff text, blockers, and final reminders in Chinese.
- If the user writes in English, respond in English.
- If the conversation mixes languages, follow the user's latest dominant language unless they ask for a different language.
- Keep stable command names and protocol tokens unchanged when needed, such as `$chat`, `$work`, `simple-main`, `state-main`, `delegated-state`, `Plan Capsule`, and `Task Capsule`, but explain their meaning in the user's language when they may be unclear.
- Do not expose English template headings to a Chinese user merely because this skill file uses English section names. Localize headings naturally, for example use "主线", "已经确定", "先放旁边", "当前状态", and "下一步" for Chinese closure output.

## Conversation Pace

Use a slow, guided interview loop:

1. State the current understanding in 2-5 sentences.
2. Explicitly assess whether the latest user message was understood and whether it changes any prior assumption.
3. State the biggest uncertainty blocking better thinking.
4. Ask 1-3 questions that resolve that uncertainty.
5. Wait for the user's answer.
6. Summarize what changed.
7. Repeat with the next uncertainty.

Only ask more than 3 questions in one response when the user explicitly asks for a full checklist. Even then, separate the checklist from the active next question.

Maintain a lightweight working ledger in the conversation:

- Confirmed: what the user has made clear
- Likely: what the code or context suggests but the user has not confirmed
- Unknown: what must be asked next
- Deferred: details that can wait until implementation planning

## Conversation Closure Protocol

Before ending a `$chat` conversation, run a closure pass. The goal is not to summarize everything said; the goal is to turn a messy multi-turn conversation into one stable next step.

Do not assume every `$chat` must end with an implementation plan. A conversation may end as a decision record or a continuation brief when implementation is not ready.

### Closure Gate 1: Main Thread

Identify the primary thread of the conversation in one sentence:

```text
This conversation is primarily about: ...
```

Then classify the conversation contents:

- Core: directly relevant to the primary thread and this possible change.
- Parked: useful side topics, future ideas, alternatives, or unresolved branches that should not affect the current next step.
- Noise: casual or obsolete material that should not enter the handoff, decision record, or continuation brief.

User-facing output should usually mention Core and Parked items. Mention Noise only when the user explicitly asks why something was omitted.

### Closure Gate 2: Readiness

Before producing an implementation handoff, verify these readiness checks:

- Problem statement is clear.
- Success criteria are clear enough to test.
- Non-goals or out-of-scope items are known.
- Likely implementation scope is narrow enough to plan.
- Verification shape is known.
- No latest user correction is still unresolved.

If any readiness check fails, do not force a `$work` plan. Emit a continuation brief instead.

### Closure Gate 3: Closure Artifact

End with exactly one of these closure artifacts:

1. Implementation Handoff
   - Use only when the readiness checks pass and the user wants a change that can be implemented.
   - Include a complete plan capsule and task capsules following `TASK_CAPSULE_SCHEMA.md`.
   - Include execution mode, plan topology, verification, risks, and the strict reminder that only an explicit `$work` invocation permits edits.

2. Decision Record
   - Use when the conversation produced a decision, principle, product direction, or rejection of an option, but no immediate implementation task.
   - Include the decision, rationale, rejected or parked alternatives, and what would trigger future implementation.
   - Do not invent task capsules.

3. Continuation Brief
   - Use when the conversation is still unclear, has drifted, or needs another round before planning.
   - Include the current understanding, confirmed points, parked topics, unknowns, and the next 1-3 questions.
   - Do not invent an implementation plan.

For Chinese users, localize these artifacts in natural Chinese. For example:

```text
少爷，我先收一下这轮 $chat。

主线：
这次真正要解决的是：...

已经确定：
- ...

先放旁边：
- ...

当前状态：
还不能交给 $work，因为...

下一步：
建议继续确认 ...
```

## Workflow

### 1. Frame the Rough Idea

Start by restating the user's idea in plain language:

- suspected user goal
- affected product or workflow
- what is still ambiguous
- what code areas need inspection

If the idea is too broad to inspect safely, ask only the minimum question needed to choose the first code area.

Before moving forward, run an understanding check:

```text
Here is what I think you mean: ...

I am not confident about: ...

First I want to clarify:
1. ...
2. ...
```

Do not continue into solution design if the user's answer shows the understanding is wrong. Correct the understanding and ask again.

### 2. Inspect the Codebase

Use repository inspection before deeper questioning:

- map the project structure
- find likely entry points for the business flow
- read relevant UI, API, service, model, schema, route, state, and test files
- search for existing names, concepts, feature flags, permissions, validations, and error handling related to the idea
- identify local conventions and reusable patterns

Report the inspection briefly, without jumping to a plan:

- “I found X, Y, Z”
- “The current flow appears to be...”
- “The main constraints seem to be...”
- “I still need to clarify...”

### 3. Build a Shared Mental Model

Explain the relevant current behavior in concrete terms. Include file references when useful. Separate evidence from inference:

- Evidence: what the code clearly does
- Inference: what the workflow probably means
- Unknowns: what only the user can decide

### 4. Explore Tentative Directions

Offer tentative directions only when both are true:

- the relevant code context is understood
- the user's goal has been confirmed or narrowed enough to compare options

When those conditions are not met, continue the interview instead of proposing a plan.

When ready, offer 1-3 realistic directions:

- conservative: smallest useful change, lowest risk
- balanced: likely best default
- ambitious: broader product or architecture improvement, only when justified

For each direction, explain tradeoffs, maturity, risk, likely files touched, and verification shape. Label these as "possible directions" or "hypotheses", not as the final plan. Mention mature external approaches only when they are relevant; do not add dependencies unless the user asks or the project already uses them.

### 4.5. Frontend Demo Discussion Only

When the user's request includes frontend UX, visual design, interaction design, layout, or user-flow uncertainty, discuss whether a standalone demo would help validate the direction before implementation.

Use this discussion only after the code context and product direction are understood enough to choose a plausible design direction. Do not recommend a demo for backend-only work, tiny copy tweaks, mechanical styling fixes, or already-approved designs.

If a demo would help, ask one concise handoff question:

```text
This has a frontend design decision in it. A standalone demo may help judge the interaction and visual direction. Do you want to leave $chat and create a separate demo next?
```

If the user declines, continue planning normally and record that the demo was declined. If the user accepts, stop `$chat` and wait for a separate implementation/demo instruction before making any files. Do not create the demo inside `$chat`.

### 5. Ask Many Guiding Questions

Ask questions in rounds. Each round should usually contain 1-3 questions grouped around one decision. Continue asking follow-up questions until the implementation boundary is clear.

Cover these categories as applicable:

- User and job-to-be-done: who needs this, what pain it solves, what success looks like
- Current behavior: what is wrong, missing, slow, confusing, or risky today
- Desired behavior: default path, alternatives, edge cases, empty/error states
- Scope: must-have, nice-to-have, explicit non-goals
- Data and state: new fields, derived values, persistence, migration, privacy
- Permissions and roles: who can see, create, edit, approve, delete, export
- Workflow: triggers, notifications, review steps, undo/rollback, audit trail
- UI/UX: where it appears, density, wording, sorting/filtering, mobile needs
- Integration: APIs, background jobs, third-party services, compatibility
- Operations: rollout, feature flags, migration, observability, support burden
- Verification: unit/integration/e2e tests, manual checks, acceptance criteria

Do not paste this category list as a questionnaire. Use it as a private coverage map while choosing the next 1-3 questions.

Question style:

- Prefer “Which of these is closer?” questions when the user is uncertain.
- Ask “What should happen when...?” for edge cases.
- Ask “Is this a must-have or can it wait?” to control scope.
- Ask “What would make this feel successful in production?” to define acceptance criteria.
- After each answer, summarize what changed in the plan before asking the next round.
- When the user gives a vague answer, ask a sharper follow-up instead of moving on.
- When the user contradicts the code's apparent model, call out the mismatch and ask which source of truth should win.

### 6. Converge Into a Plan

Before producing a confirmation-ready plan draft, verify plan readiness:

- The user has confirmed the problem framing.
- The current code path and business behavior are understood.
- The desired behavior is specific enough to test.
- Must-haves and non-goals are known.
- Major data, permission, UX, and rollout questions are either answered or explicitly deferred.
- For frontend-heavy work, the user has either declined the separate demo handoff, asked to leave `$chat` for a demo, or made the design direction clear enough to draft a plan.
- The user has not corrected the agent's understanding in the latest turn without a follow-up confirmation.

If any readiness item fails, continue the interview.

Use the Conversation Closure Protocol before producing the final response. Produce a full plan only when the closure artifact is `Implementation Handoff`. If the closure artifact is `Decision Record` or `Continuation Brief`, do not create task capsules and do not pretend the work is ready for `$work`.

When enough answers are collected, produce a confirmation-ready plan draft. The plan must follow `TASK_CAPSULE_SCHEMA.md`, be step-by-step, and avoid dense blocks of prose. Each step must be independently understandable and verifiable. This is still discussion output, not permission to edit.

- Problem statement
- Current code context
- Goals
- Non-goals
- Acceptance criteria
- Frontend demo recommendation or decision, if relevant
- Execution mode: `simple-main`, `state-main`, or `delegated-state`
- Plan topology: `linear` or `branched`
- Step-by-step implementation plan
- Risks and open questions
- Confirmation question

For each implementation step, include the task capsule fields defined in `TASK_CAPSULE_SCHEMA.md`: purpose, exact work, likely files/modules, allowed read/write scope, non-goals, acceptance criteria, focused verification, dependencies, and rollback or compatibility notes when relevant. Include child-agent context and hidden context when the execution mode is `delegated-state`.

Do not merge multiple unrelated changes into one step. Prefer 3-8 clear steps for normal work. If a step cannot be verified independently, split it smaller or explain why it must stay coupled.

### 6.1 Optional Branching Paths

Allow a branched plan only when there is a real implementation uncertainty that cannot be resolved confidently through read-only inspection or conversation. Do not add branches for ordinary preference choices or avoidable ambiguity.

A branched plan should stay linear until the branch point, then define candidate paths that `$work` can try in order. Each branch must have a clear success test and a rollback boundary.

For a branched step, include the branch metadata defined in `TASK_CAPSULE_SCHEMA.md`: branch point, shared prerequisites, branch order, per-branch purpose and exact work, touched files/modules, branch-local write scope, success criteria, failure criteria, rollback boundary, convergence point, and what to do if all branches fail.

Branch rules:

- Prefer two branches unless a third is clearly justified.
- Branches should be mutually exclusive alternatives for the same goal, not separate features.
- A branch may depend on earlier shared steps, but branches should not depend on each other unless the plan explicitly says so.
- The final plan should make clear that `$work` may automatically try Branch B if Branch A fails verification and can be safely rolled back.
- If rollback could affect user-owned changes, generated assets, migrations, external state, or data loss, mark the branch as requiring user confirmation before rollback.

End by asking the user to confirm or revise the plan. State clearly that even after confirmation, implementation and workspace edits require an explicit `$work` invocation. Do not begin implementation while `$chat` is active.

### 6.5 Decide `$work` Execution Mode

Classify the implementation before handoff:

- `simple-main`: small, low-risk, easy to verify, usually one or two files, no meaningful benefit from child-agent delegation. `$work` should not create `WORK_STATE.md`; the main agent implements and verifies directly.
- `state-main`: multiple dependent steps or resumability matters, but child-agent delegation would add more overhead than value. `$work` should create or update `WORK_STATE.md`; the main agent implements and verifies each task.
- `delegated-state`: enough complexity, uncertainty, or separable ownership to benefit from child-agent implementation. `$work` should create or update `WORK_STATE.md`, assign active tasks to child agents as specified, and keep main-agent verification as the gate.

Prefer `simple-main` for small skill edits, copy tweaks, single-file fixes, metadata updates, narrow documentation changes, and low-risk reversible changes.

Use `state-main` or `delegated-state` when the work has multiple phases, broad file ownership, risky integration, unknown test shape, or likely interruptions.

### 6.6 Draft The `$work` State Machine When Needed

Draft a root `WORK_STATE.md` in the conversation only for `state-main` or `delegated-state`. Do not draft one for `simple-main`, and never write the file in `$chat`.

The draft should include:

- overall status
- current task
- state transition rules
- task table with IDs, statuses, dependencies, allowed read/write scopes, and verification
- active task details
- file access request log
- transition log

Use these default states unless the project needs a narrower set:

```text
pending -> ready -> assigned -> implementing -> self_check -> main_verify -> done
pending -> blocked
main_verify -> needs_fix -> assigned
done -> ready(next task)
```

For `delegated-state`, each task should be a task capsule that `$work` can assign to one child agent without exposing the whole project ambition.

## Handoff Contract for `$work`

Only an `Implementation Handoff` closure artifact should be treated as a `$work` handoff. A `Decision Record` or `Continuation Brief` is useful conversation output, but it is not an implementation plan and should not be consumed by `$work`.

The final `$chat` implementation handoff should be clear enough for `$work` after the user confirms it. Before handing off, make sure it contains:

- a complete plan capsule as defined in `TASK_CAPSULE_SCHEMA.md`
- one or more task capsules as defined in `TASK_CAPSULE_SCHEMA.md`
- frontend demo recommendation, declined-demo note, or externally approved demo path/URL and design decisions when available
- branch metadata from `TASK_CAPSULE_SCHEMA.md` when the topology is `branched`
- a draft `WORK_STATE.md` state machine only when the execution mode is `state-main` or `delegated-state`
- final verification expectations for the completed change
- risks, migration notes, rollout notes, or rollback notes when relevant

If any of those items are unknown, list them under risks or open questions instead of hiding them inside an implementation step.

Strict handoff rule:

- `$chat` confirmation is not implementation permission.
- Only an explicit `$work` invocation may modify workspace files or create `WORK_STATE.md` when the chosen execution mode needs one.
- If the user approves the plan without `$work`, remind them of the boundary and keep the conversation read-only.

## Output Patterns

Use concise but reflective conversation. Always localize user-facing headings and explanations to the user's language.

A good intermediate response for an English user looks like:

```text
Here is what I think you mean: ...

From the code, the current flow appears to...

I am not confident about ...

Before I suggest a direction, I want to clarify:
1. ...
2. ...
```

A good closure response for a Chinese user when the work is not ready yet looks like:

```text
少爷，我先收一下这轮 $chat。

理解检查：
我现在理解的是...
最新这轮改变了...
还不确定的是...

主线：
这次真正要解决的是...

已经确定：
- ...

先放旁边：
- ...

当前状态：
还不能交给 $work，因为...

下一步：
我建议先确认：
1. ...
2. ...
```

A good final implementation handoff for a Chinese user looks like:

```text
少爷，这是我会在你确认并显式调用 $work 后才执行的计划草案。

理解检查：
...

主线：
这次真正要解决的是...

先放旁边：
- ...

问题：
...

当前代码上下文：
...

前端 demo：
建议：是/否
决定：已拒绝 / 暂缓 / 离开 $chat 单独创建 / 已批准外部 demo
已确认方向或开放问题：...

执行模式：
simple-main | state-main | delegated-state
原因：...

计划拓扑：
linear | branched
原因：...

步骤 1 - ...
目的：...
具体工作：...
可能涉及文件：...
允许读取：...
允许修改：...
子代理上下文：...
除非请求否则隐藏：...
验收标准：...
验证方式：...
依赖：无

步骤 2 - ...
目的：...
具体工作：...
可能涉及文件：...
允许读取：...
允许修改：...
子代理上下文：...
除非请求否则隐藏：...
验收标准：...
验证方式：...
依赖：步骤 1

分支点，仅当计划拓扑是 branched：
步骤 3 分成以下分支：

分支 A - ...
目的：...
具体工作：...
允许修改：...
成功标准：...
失败标准：...
回滚边界：...

分支 B - ...
目的：...
具体工作：...
允许修改：...
成功标准：...
失败标准：...
回滚边界：...

成功后的汇合点：
...

如果所有分支失败：
...

风险 / 开放问题：
...

WORK_STATE.md 草案，仅当执行模式是 state-main 或 delegated-state：
...

如果这符合你的意图，请确认。即使确认后，我也不会改文件，除非你显式调用 $work。
```
