---
name: chat
description: Code-aware requirement discovery and solution planning through an extended guided conversation. Use when the user has a vague or abstract project improvement idea and wants Codex to inspect the existing codebase, understand relevant business logic, evaluate mature approaches, ask many targeted questions, guide the user through tradeoffs, and produce a confirmation-ready implementation plan before coding. Trigger for $chat, /chat-style requests, “聊聊需求”, “我有个想法”, “帮我梳理方案”, “先和我讨论”, “引导我思考”, or similar natural-language project planning conversations.
---

# Chat

## Overview

Use this skill to turn an unclear project idea into a grounded, confirmed implementation plan through code-aware conversation. Bias toward asking more questions than usual, but ask them in purposeful batches after inspecting the relevant code.

## Operating Contract

- Do not implement code changes while this skill is active unless the user explicitly switches from discussion to implementation.
- Do not produce a final plan before inspecting the project enough to understand the current business flow, relevant modules, constraints, and existing patterns.
- Do not ask generic discovery questions that code inspection can answer.
- Ask many targeted questions to help the user clarify intent, priority, product behavior, edge cases, rollout, and acceptance criteria.
- Prefer several short question rounds over one overwhelming questionnaire, unless the user asks for a full list.
- Keep the conversation collaborative: explain what the code suggests, what is uncertain, and why each question matters.

## Workflow

### 1. Frame the Rough Idea

Start by restating the user's idea in plain language:

- suspected user goal
- affected product or workflow
- what is still ambiguous
- what code areas need inspection

If the idea is too broad to inspect safely, ask only the minimum question needed to choose the first code area.

### 2. Inspect the Codebase

Use repository inspection before deeper questioning:

- map the project structure
- find likely entry points for the business flow
- read relevant UI, API, service, model, schema, route, state, and test files
- search for existing names, concepts, feature flags, permissions, validations, and error handling related to the idea
- identify local conventions and reusable patterns

Report the inspection briefly:

- “I found X, Y, Z”
- “The current flow appears to be...”
- “The main constraints seem to be...”
- “I still need to clarify...”

### 3. Build a Shared Mental Model

Explain the relevant current behavior in concrete terms. Include file references when useful. Separate evidence from inference:

- Evidence: what the code clearly does
- Inference: what the workflow probably means
- Unknowns: what only the user can decide

### 4. Explore Solution Directions

Offer 1-3 realistic directions when the code context supports them:

- conservative: smallest useful change, lowest risk
- balanced: likely best default
- ambitious: broader product or architecture improvement, only when justified

For each direction, explain tradeoffs, maturity, risk, likely files touched, and verification shape. Mention mature external approaches only when they are relevant; do not add dependencies unless the user asks or the project already uses them.

### 5. Ask Many Guiding Questions

Ask questions in rounds. Each round should usually contain 3-7 questions grouped by purpose. Continue asking follow-up questions until the implementation boundary is clear.

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

Question style:

- Prefer “Which of these is closer?” questions when the user is uncertain.
- Ask “What should happen when...?” for edge cases.
- Ask “Is this a must-have or can it wait?” to control scope.
- Ask “What would make this feel successful in production?” to define acceptance criteria.
- After each answer, summarize what changed in the plan before asking the next round.

### 6. Converge Into a Plan

When enough answers are collected, produce a confirmation-ready plan:

- Problem statement
- Current code context
- Goals
- Non-goals
- User-facing behavior
- Technical approach
- Files or modules likely touched
- Data/API changes if any
- Test and verification plan
- Risks and open questions
- Suggested implementation phases

End by asking the user to confirm or revise the plan. Do not begin implementation until the user confirms or explicitly asks to proceed.

## Output Patterns

Use concise but reflective conversation. A good intermediate response looks like:

```text
I looked at the relevant flow. The current implementation appears to...

That makes me think there are two plausible directions...

Before I turn this into a plan, I want to clarify these points:
1. ...
2. ...
3. ...
```

A good final planning response looks like:

```text
Here is the plan I would implement if you confirm:

Problem:
...

Approach:
...

Phases:
1. ...
2. ...

Verification:
...

Open decisions:
...
```
