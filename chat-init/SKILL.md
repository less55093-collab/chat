---
name: chat-init
description: Initialize a new or existing project through multi-role requirement discovery, similar-project research, document-set selection, and confirmed project documentation scaffolding. Use when the user invokes $chat-init or /chat-init, asks to initialize a project, wants AI roles to interview them before project setup, wants project architecture/API/database/requirements docs generated from discussion, or wants GitHub alternatives evaluated before starting from scratch.
---

# Chat Init

## Overview

Use this skill to turn a rough project idea into a confirmed set of initialization documents. Run a structured multi-role interview, research similar GitHub projects when relevant, recommend the right documents for the project shape, ask for confirmation, then create or update only the confirmed documentation.

This skill initializes project knowledge and planning artifacts. It does not scaffold application code, install dependencies, run migrations, or adopt a third-party project unless the user explicitly asks after the initialization decision.

## Operating Contract

- Inspect the current repository before asking deep questions, unless the user is clearly starting in an empty folder.
- Preserve existing work. Check the worktree before edits and never overwrite existing documents without explicit confirmation.
- Do not generate documents until the user confirms the initialization brief.
- Ask focused questions in rounds. Prefer 1-3 high-leverage questions per round instead of a long questionnaire.
- Each role must contribute at least one pass. If a role is not applicable, record that finding instead of forcing irrelevant questions.
- Every role may make suggestions, but suggestions must be labeled as recommendations, not decisions.
- Treat GitHub research as advisory. Search for comparable projects, evaluate fit and license, and ask the user whether to use any candidate as inspiration or a base. Do not clone, vendor, or run external code without explicit user confirmation.
- Generate only useful documents. Avoid empty template files that contain no project-specific decisions.
- If the user asks to initialize code as well as docs, finish the documentation initialization first, then hand off to a separate implementation plan or `$work`.

## Workflow

### 1. Start With A Repo-Aware Snapshot

Inspect:

- top-level files and directories
- existing docs, specs, ADRs, README files, or planning artifacts
- package/framework clues when present
- existing skill/plugin structure if the repository is itself a Codex skill
- git status before editing

Report briefly:

- what the repository appears to be
- whether this is a new project, existing project, or skill/plugin
- where documents should likely live
- what is still unknown

If the repository has existing documentation conventions, follow them. Otherwise default to `docs/`.

### 2. Clarify The Project Seed

Ask the user for the smallest useful project seed if it is missing:

- target user or operator
- core problem
- intended product type
- must-have outcome
- important constraints

Do not wait for perfect detail. The multi-role interview can uncover the rest.

### 3. Run Multi-Role Discovery

Use role passes as lenses, not theater. Each pass should either ask a question, make a concrete suggestion, or declare itself not applicable with a reason.

Required role passes:

- Product: users, jobs-to-be-done, MVP boundary, success criteria.
- Architecture: technical shape, modules, integration points, deployment assumptions.
- UX / Workflow: user journey, key screens or commands, error/empty states, handoffs.
- Data: entities, persistence, privacy, migration needs. Mark not applicable for stateless tools.
- API / Integration: public/internal APIs, external services, authentication, compatibility. Mark not applicable when there is no interface contract.
- Security / Operations: secrets, permissions, abuse cases, observability, rollout, support burden.
- Testing: acceptance criteria, unit/integration/e2e/manual verification, fixtures.
- Innovation: non-obvious improvements, alternative product angles, automation opportunities. Separate near-term from later bets.
- GitHub Research: comparable projects, libraries, or templates, with license/activity/tech-stack fit and a recommendation on whether to reuse, fork, or only learn from them.

Pace:

- Start with Product, Architecture, and UX.
- Bring Data and API only after the product shape is clear enough.
- Bring Security/Ops and Testing before finalizing requirements.
- Run Innovation after the MVP boundary is visible, so it can challenge the default path without derailing discovery.
- Run GitHub Research after the core idea is specific enough to search effectively.

### 4. Similar-Project Research

When researching similar projects:

- Use current web/GitHub search or `gh` if configured.
- Prefer repositories with compatible licenses, recent activity, clear docs, and a matching tech stack.
- Summarize 3-5 candidates when available.
- Include why each candidate is relevant, why it may not fit, license notes, and whether it is better as a base, reference, or rejection.
- Ask the user whether any candidate should influence the project before documents are generated.

If network access or GitHub access is unavailable, record the research as an open item and continue with local initialization.

### 5. Recommend The Document Set

Always consider these core documents:

- `PROJECT_BRIEF.md`: problem, audience, value proposition, constraints.
- `PRD.md`: goals, non-goals, requirements, user flows, edge cases.
- `ARCHITECTURE.md`: system shape, module boundaries, core decisions.
- `ACCEPTANCE_CRITERIA.md`: user-visible acceptance tests and completion definition.
- `DECISIONS.md`: decision log with rejected alternatives.

Generate these only when the project needs them:

- `API.md`: endpoints, schemas, auth, errors, versioning, examples.
- `DATABASE.md`: entities, relationships, indexes, migrations, retention.
- `UX_FLOWS.md`: screens, states, command flows, navigation, copy notes.
- `INTEGRATIONS.md`: third-party services, webhooks, rate limits, credentials.
- `SECURITY.md`: permissions, threat notes, sensitive data, abuse prevention.
- `OPERATIONS.md`: deploy, config, observability, support, rollback.
- `TEST_PLAN.md`: deeper test strategy when the project has meaningful implementation risk.
- `ROADMAP.md`: milestones and sequencing when scope exceeds one implementation pass.

If the repository already has equivalent files, update or extend those names instead of creating duplicates.

### 6. Confirmation Gate

Before editing files, present an initialization brief:

- Confirmed project understanding
- Open questions that still matter
- Role-pass summary and recommendations
- Similar GitHub candidates and reuse recommendation
- Proposed document list with paths
- Explicit non-goals for this initialization
- Any existing files that would be updated

Ask the user to confirm, revise, or remove documents from the list. Do not continue until confirmation is clear.

### 7. Generate Documents

After confirmation:

- Create the confirmed documents using the repository's existing style when available.
- Keep documents specific to the user's answers and role findings.
- Mark unresolved items in a clearly named "Open Questions" section.
- Include decision records for major choices and rejected alternatives.
- Cross-link related documents when useful.
- Avoid boilerplate sections with no content.
- Do not create implementation files, package manifests, schemas, migrations, or generated app code unless the user explicitly expands scope beyond chat-init.

Preferred doc tone:

- concise, actionable, and project-specific
- clear enough for `$work` or a future implementation agent to execute
- honest about assumptions and unknowns

### 8. Validate And Report

Before final response:

- Re-read generated or modified documents.
- Check that confirmed answers are reflected and contradictions are not silently hidden.
- Check that optional docs were only created when justified.
- Run lightweight validation appropriate to docs, such as markdown lint if already available or a manual structure check.
- Inspect git diff and summarize changed files.

Final response must include:

- documents created or updated
- key decisions captured
- remaining open questions
- verification performed
- suggested next handoff, usually `$work` only after the user confirms they want implementation

## Output Shape

During discovery:

```text
Current understanding:
...

Role pass - Product:
Recommendation: ...
Question: ...

Role pass - Architecture:
Recommendation: ...
Question: ...
```

Before editing:

```text
Initialization brief:
Confirmed: ...
Open questions: ...
GitHub candidates: ...
Proposed documents:
- docs/PROJECT_BRIEF.md
- docs/PRD.md
Non-goals: ...

Please confirm or revise this document set before I create files.
```

After editing:

```text
Initialized:
- docs/PROJECT_BRIEF.md
- docs/PRD.md

Captured decisions:
- ...

Verification:
- Re-read generated docs
- Checked git diff

Remaining open questions:
- ...
```
