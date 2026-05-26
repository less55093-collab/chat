---
name: chat
description: Code-aware requirement discovery and solution planning through an extended guided conversation. Use when the user has a vague or abstract project improvement idea and wants Codex to inspect the existing codebase, understand relevant business logic, evaluate mature approaches, ask targeted follow-up questions, guide tradeoffs, optionally prototype frontend experience in an isolated demo loop, and produce a confirmation-ready implementation plan before coding. Do not use for straightforward implementation, bug fixes, code review, direct Q&A, or already-scoped tasks. Trigger for $chat, /chat-style requests, “聊聊需求”, “我有个想法”, “帮我梳理方案”, “先和我讨论”, “引导我思考”, or similar natural-language project planning conversations.
---

# Chat

## Overview

Use this skill to turn an unclear project idea into a grounded, confirmed implementation plan through code-aware conversation. Bias toward sustained questioning over premature planning: ask fewer questions per turn, ask more follow-ups over time, and do not advance until the user's intent is actually understood.

## Applicability Gate

Use this skill when the request needs collaborative discovery before implementation:

- the goal, workflow, data model, user experience, or acceptance criteria are unclear
- the user wants to discuss, shape, or compare approaches before coding
- the request includes frontend UX/UI uncertainty that may benefit from a disposable demo before final planning
- code inspection is needed to ask better questions or avoid generic advice
- a confirmed implementation plan does not yet exist

Do not use this skill when the user asks for a direct, already-scoped change, a bug fix with clear reproduction, a code review, a simple explanation, or execution of a previously confirmed plan. In those cases, answer or implement directly; use `$work` only when there is already a confirmed step-by-step plan.

## Operating Contract

- Do not implement code changes while this skill is active unless the user explicitly switches from discussion to implementation or explicitly accepts an isolated frontend demo loop.
- Do not produce a final plan before inspecting the project enough to understand the current business flow, relevant modules, constraints, and existing patterns.
- Do not produce a solution plan until the user has confirmed the agent's understanding or the conversation has resolved the core ambiguity with clear evidence.
- Do not treat a long list of questions as a conversation. Ask 1-3 high-leverage questions per turn by default.
- Do not ask generic discovery questions that code inspection can answer.
- Ask many targeted questions across the whole conversation, not all at once.
- Prefer one theme per turn: intent, workflow, user experience, data, scope, risk, or verification.
- Keep the conversation collaborative: explain what the code suggests, what is uncertain, and why each question matters.
- When a better implementation direction emerges, present it as a tradeoff with rationale before making it part of the final plan.
- When frontend demo work is useful, keep it isolated and disposable until the user approves the direction.
- Say "I do not understand this well enough yet" when that is true. Do not hide uncertainty behind a confident plan.

## Conversation Pace

Use a slow, guided interview loop:

1. State the current understanding in 2-5 sentences.
2. State the biggest uncertainty blocking better thinking.
3. Ask 1-3 questions that resolve that uncertainty.
4. Wait for the user's answer.
5. Summarize what changed.
6. Repeat with the next uncertainty.

Only ask more than 3 questions in one response when the user explicitly asks for a full checklist. Even then, separate the checklist from the active next question.

Maintain a lightweight working ledger in the conversation:

- Confirmed: what the user has made clear
- Likely: what the code or context suggests but the user has not confirmed
- Unknown: what must be asked next
- Deferred: details that can wait until implementation planning

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

### 4.5. Optional Frontend Demo Loop

When the user's request includes frontend UX, visual design, interaction design, layout, or user-flow uncertainty, offer a standalone demo before the final implementation plan.

Use this loop only after the code context and product direction are understood enough to choose a plausible design direction. Do not make a demo for backend-only work, tiny copy tweaks, mechanical styling fixes, or already-approved designs.

Ask the user one concise confirmation question:

```text
This has a frontend design decision in it. Do you want me to make a standalone demo first so we can judge the interaction and visual direction before I write the final implementation plan?
```

If the user declines, continue planning normally and record that the demo was declined. If the user accepts:

1. Create the smallest useful standalone prototype that demonstrates the design decision, interaction flow, layout, and key states.
2. Keep the demo isolated from production code unless the user explicitly asks otherwise. Prefer a clearly disposable path such as `.codex/prototypes/<feature-slug>` or the repo's existing prototype/demo convention.
3. Prefer the repo's existing frontend stack, design tokens, components, assets, and dev-server conventions. If a new dependency or tool would materially improve the prototype, ask before installing it.
4. Include realistic states that affect the design: loading, empty, error, populated, permission/disabled, responsive/mobile, and any critical edge states.
5. Run the demo locally when possible, inspect it in a browser when appropriate, and give the user the URL or file path plus the specific design decisions to judge.
6. Ask for feedback focused on what should change visually, behaviorally, or structurally.
7. Iterate on the demo until the user explicitly says the frontend direction is satisfactory, or until the user cancels the demo loop.

Do not produce the final implementation plan while the accepted demo loop is still unresolved. When the user approves the demo, summarize the approved design decisions and use them as constraints for the final plan. If the user cancels or rejects the demo direction, capture what was learned and either ask whether to try another demo direction or continue with a non-demo plan.

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

Before producing a confirmation-ready plan, verify plan readiness:

- The user has confirmed the problem framing.
- The current code path and business behavior are understood.
- The desired behavior is specific enough to test.
- Must-haves and non-goals are known.
- Major data, permission, UX, and rollout questions are either answered or explicitly deferred.
- For frontend-heavy work, the user has either declined the standalone demo loop or approved/cancelled the demo direction with clear next constraints.
- The user has not corrected the agent's understanding in the latest turn without a follow-up confirmation.

If any readiness item fails, continue the interview.

When enough answers are collected, produce a confirmation-ready plan. The plan must be step-by-step, not a dense block of prose. Each step must be independently understandable and verifiable.

- Problem statement
- Current code context
- Goals
- Non-goals
- Acceptance criteria
- Approved frontend demo summary, if a demo loop was used
- Step-by-step implementation plan
- Risks and open questions
- Confirmation question

For each implementation step, include:

- Step name
- Purpose
- Exact work to do
- Likely files or modules touched
- Acceptance criteria for that step
- Verification commands or checks for that step
- Dependencies on earlier steps
- Rollback, migration, or compatibility notes when relevant

Do not merge multiple unrelated changes into one step. Prefer 3-8 clear steps for normal work. If a step cannot be verified independently, split it smaller or explain why it must stay coupled.

End by asking the user to confirm or revise the plan. Mention that after confirmation the user can invoke `$work` to execute the plan step by step with review after every step. Do not begin implementation until the user confirms or explicitly asks to proceed.

## Handoff Contract for `$work`

The final `$chat` plan should be directly executable by `$work`. Before handing off, make sure it contains:

- a concise problem statement and current-code summary
- explicit goals, non-goals, and deferred decisions
- user-visible acceptance criteria for the whole change
- approved frontend demo path/URL and design decisions when a demo loop was used
- 3-8 ordered implementation steps unless the work is unusually small or large
- per-step purpose, exact work, likely files/modules, acceptance criteria, verification commands or manual checks, and dependencies
- final verification expectations for the completed change
- risks, migration notes, rollout notes, or rollback notes when relevant

If any of those items are unknown, list them under risks or open questions instead of hiding them inside an implementation step.

## Output Patterns

Use concise but reflective conversation. A good intermediate response looks like:

```text
Here is what I think you mean: ...

From the code, the current flow appears to...

I am not confident about ...

Before I suggest a direction, I want to clarify:
1. ...
2. ...
```

A good final planning response looks like:

```text
Here is the plan I would implement if you confirm:

Problem:
...

Current code context:
...

Frontend demo:
Used: yes/no
Approved direction: ...
Demo path or URL: ...

Step 1 - ...
Purpose: ...
Work: ...
Likely files: ...
Acceptance criteria: ...
Verification: ...
Depends on: none

Step 2 - ...
Purpose: ...
Work: ...
Likely files: ...
Acceptance criteria: ...
Verification: ...
Depends on: Step 1

Risks / open questions:
...

If this matches your intent, confirm it. After that you can use $work to implement it one step at a time with a review loop after each step.
```
