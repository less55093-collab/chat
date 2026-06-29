---
name: chat-deep
description: Research-enabled requirement discovery before implementation. Use when the user invokes $chat-deep or /chat-deep, asks to deeply research a project idea before coding, wants Codex to decide research depth from task complexity, needs web-wide solution discovery, benchmark/market/prior-art research, architecture or vendor comparison, or asks to install research tools, skills, plugins, MCP servers, CLIs, or APIs to improve research breadth, depth, accuracy, citation quality, or report quality. For tool, integration, automation, CLI, MCP, plugin, or API needs, this skill defaults to reuse-first discovery across GitHub, package registries, official ecosystems, templates, and comparable implementations before recommending custom development, and must carry adopted reusable candidates into the $work handoff instead of silently rebuilding equivalent functionality. This skill maintains a persistent markdown research state machine so long research survives context compression and does not repeat completed work. It may install or configure research-only tooling after explicit user confirmation and may write markdown research briefs, but must not edit product/source implementation files; implementation still requires an explicit $work invocation.
---

# Chat Deep

## Overview

Use this skill as the research execution bridge between `$chat` and `$work`. `$chat-deep` turns an uncertain user need into a sourced markdown research brief, a practical solution strategy, and a `$work`-ready implementation handoff.

The default center of gravity is a user's new requirement, not an existing codebase. Inspect local project context when the user provides one or when implementation fit depends on it, but do not let repository analysis displace the main job: understand the user's need, discover existing wheels or industry solutions, and explain which path should be reused or adopted.

For tool-like requests, `$chat-deep` is reuse-first. Before designing a new implementation, it should actively look for existing projects, libraries, CLIs, MCP servers, plugins, SaaS APIs, hosted products, official APIs, standards, protocols, industry workflows, templates, and proven implementation examples that can be used directly, wrapped, forked, adapted, or adopted as the solution pattern. Custom development is the recommendation only after the research explains why existing options do not satisfy the user's core constraints.

Reuse-first must survive the handoff. If a candidate is marked `adopt`, or if a `shortlist` candidate is the preferred direction, the recommended solution and `$work` task capsules must be framed around using that candidate. Do not convert the research into a from-scratch implementation plan unless all viable candidates are rejected or deferred with explicit evidence.

`$chat-deep` is more permissive than `$chat`, but only for research. It may install or configure research-related skills, plugins, MCP servers, CLIs, packages, and API clients after explicit user confirmation. It must not modify product code, migrations, production data, app assets, commits, branches, or deployment state.

## Permission Boundary

Allowed after explicit user confirmation:

- Install, update, or configure research-only tools when they materially improve research depth, breadth, accuracy, source access, citation quality, or report quality.
- Use package managers, CLIs, MCP setup commands, browser automation, API clients, or skill/plugin installers only for research tooling.
- Write markdown research artifacts and persistent research state under a clearly research-scoped path, preferably `.codex/research/` in the active workspace when available.
- Create temporary caches or tool state required for research, while avoiding product build artifacts and application state.

Not allowed:

- Edit source files, product docs, app assets, migrations, generated application files, lockfiles for the target project, or configuration that changes app behavior.
- Run migrations, seed databases, mutate production services, commit, push, create pull requests, or deploy.
- Treat the research brief or user approval as permission to implement. Implementation requires an explicit `$work` invocation.
- Install broad or unrelated tooling "just in case"; every tool must have a research-specific reason.
- Rely on conversation history as the source of truth when persistent research state files exist.

If a research tool installation would modify the target repository's dependency files or lockfiles, install it outside the target project or stop and ask for a separate location.

## Persistent Research State

`$chat-deep` must be resumable. For any `standard`, `deep`, or `exhaustive` run, create or resume a research run directory after explicit user confirmation. Prefer this layout:

```text
.codex/research/YYYY-MM-DD-topic/
  RESEARCH_STATE.md
  SOURCES.md
  FINDINGS.md
  DECISIONS.md
  RESEARCH_BRIEF.md
```

Use existing paths when the user provides one or when the active conversation already named a research run. Otherwise propose a concise slug and create the directory only after the user confirms the research plan.

### Required State Files

`RESEARCH_STATE.md` is the source of truth after it exists. It must include:

- `Current state`: one state from the state machine.
- `User need`: the current understanding of the request.
- `Research depth`: `light`, `standard`, `deep`, or `exhaustive`.
- `Scope`: included questions, excluded questions, source constraints, and output path.
- `Done`: completed phases and concrete artifacts produced.
- `Next action`: the next single action or phase.
- `Stop condition`: what proves the current phase can advance.
- `No-repeat ledger`: completed queries, sources, tools, and extraction attempts that should not be repeated.
- `Open questions`: unresolved decisions or missing credentials.
- `State log`: timestamped transitions with one-line rationale.

`SOURCES.md` records every useful source and every rejected source. Include URL or local path, source type, date accessed, quality note, relevant claims, and whether it was adopted, rejected, or needs follow-up.

For reuse-first research, `SOURCES.md` must also record candidate projects or packages that were rejected, including the reason: stale maintenance, incompatible license, weak docs, poor fit, missing capability, risky dependency surface, or unclear adoption path.

`FINDINGS.md` records synthesized claims. Each finding should include supporting sources, confidence, project relevance, and any contradiction or caveat.

`DECISIONS.md` records recommended directions, rejected alternatives, deferred questions, and the rationale for each decision.

`RESEARCH_BRIEF.md` is the final user-facing and `$work`-facing brief. It must be assembled from `RESEARCH_STATE.md`, `SOURCES.md`, `FINDINGS.md`, and `DECISIONS.md`, not by restarting research.

### State Machine

Use this state machine unless the user asks for a narrower process:

```text
intake -> interviewing -> framed -> scoped -> plan_confirmed -> tooling_ready -> collecting -> extracting -> synthesizing -> validating -> handoff_ready -> complete
any -> blocked
validating -> collecting
synthesizing -> extracting
handoff_ready -> validating
```

State meanings:

- `intake`: understand the initial request and determine whether `$chat-deep` is appropriate.
- `interviewing`: run a focused, multi-round discovery interview to surface the user's real goal, background, constraints, decision criteria, and success definition.
- `framed`: the user has confirmed or corrected a concrete problem framing that is stable enough to scope research.
- `scoped`: define research questions, depth, candidate tools, artifact path, and confirmation request.
- `plan_confirmed`: user has approved the research plan and state directory can be created or updated.
- `tooling_ready`: required research tools are available, installed, configured, skipped, or explicitly blocked.
- `collecting`: search, crawl, inspect repositories, collect official docs, or gather raw sources.
- `extracting`: read and extract relevant evidence from collected sources.
- `synthesizing`: turn evidence into findings, patterns, options, and tradeoffs.
- `validating`: check source quality, contradictions, stale facts, project fit, and coverage gaps.
- `handoff_ready`: final recommendation and `$work` task capsules are ready for review.
- `complete`: user-facing closeout has been delivered and the brief path is known.
- `blocked`: progress requires missing credentials, unavailable tools, user scope decisions, or external state.

### Resume Protocol

At the start of every `$chat-deep` turn:

1. Locate the active research run directory if one exists.
2. Read `RESEARCH_STATE.md` before doing new research.
3. Read only the necessary supporting files for the current state: `SOURCES.md`, `FINDINGS.md`, `DECISIONS.md`, or `RESEARCH_BRIEF.md`.
4. State whether this is a new run or a resumed run.
5. Continue from `Next action`; do not restart completed phases unless `RESEARCH_STATE.md` says the previous result failed validation.

If state files conflict with chat history, trust the state files and ask the user before overwriting them.

### Anti-Loop Rules

- Before every search or source extraction, check the `No-repeat ledger` and `SOURCES.md`.
- Do not repeat the same query, source, tool call, or extraction attempt unless there is a written reason, such as new scope, stale result, failed extraction, or contradictory evidence.
- If a phase fails twice for the same reason, transition to `blocked` or ask a focused question instead of trying the same action again.
- If validation finds a gap, write the exact gap and the minimum next search or extraction needed before returning to `collecting` or `extracting`.
- Never erase completed findings to simplify the narrative; mark them rejected or superseded in `DECISIONS.md`.

## Workflow

### 0. Resume Or Create State

For `standard`, `deep`, and `exhaustive` research, start with the Persistent Research State protocol. If this is a new run, include the proposed state directory in the research plan and create initial state files only after user confirmation. For `light` research, persistent state is optional unless the work spans more than one turn.

### 1. Interview And Frame The Request

Do not rush from an initial user prompt into a research plan. `$chat-deep` must treat requirement discovery as a first-class phase because many users do not initially express the real problem, constraints, or decision context clearly enough for useful research.

Start by restating the user's initial request, the new capability or workflow they appear to want, and why research may matter. If an existing project is relevant and available, inspect it enough to understand fit constraints; otherwise keep the focus on the user need and the external solution landscape.

Run a focused, multi-round intake interview before proposing a research plan. Ask several small batches of high-value questions, usually 3-6 questions per round, and continue until the problem framing and objective project situation model are stable enough that the eventual research plan is likely to answer the user's actual need rather than a guessed version of it. Prefer concrete questions over broad prompts.

The situation model is not psychological analysis of the user. Do not infer hidden emotions, motives, or personal state, and do not claim to completely understand the user. Focus on project-relevant context that changes the right research question, candidate evaluation criteria, or implementation handoff. A small amount of empathy is fine when it helps the user express context, but the interview should remain objective and decision-focused.

The interview should uncover:

- The user's real goal and the decision they need the research to support.
- The project stage, background, users, workflow, business or technical context, and why this matters now.
- Existing systems, codebases, tools, vendors, processes, or prior attempts that constrain the answer.
- Hard constraints such as budget, timeline, platform, language, security, compliance, data access, deployment, licensing, rollout limits, migration tolerance, support burden, or team skill.
- Affected users, maintainers, operators, reviewers, admins, downstream systems, and ownership boundaries.
- Prior attempts, rejected approaches, workarounds, incidents, and why they failed or are insufficient.
- Success criteria, failure criteria, must-haves, nice-to-haves, and explicit non-goals.
- The cost of choosing poorly and what evidence would make a recommendation trustworthy.
- The user's current assumptions, preferences, and options they are already considering.
- Expected output shape: brief, comparison, architecture recommendation, vendor shortlist, implementation handoff, risk analysis, or another artifact.

When the user is unsure, help them answer by offering hypotheses, examples, option sets, and tradeoffs for them to confirm or correct. Separate user-confirmed facts, source or code evidence, and agent assumptions. Ask follow-up questions when answers reveal contradictions, missing context, ambiguous priorities, or hidden project constraints. Do not ask filler questions; every question should reduce research ambiguity.

Before proposing the research plan, write a concise "problem framing and project situation" summary in the user's language and ask the user to confirm or correct it. Only move from `interviewing` to `framed` after the user confirms the framing or gives enough correction that the framing is effectively confirmed.

Always state:

- What is already understood.
- What remains uncertain.
- Whether the request needs external research or can stay in normal `$chat`.
- The proposed research depth.
- Whether the request is tool-like and therefore requires reuse-first discovery before custom design.
- Whether the problem framing has been confirmed by the user.

### 2. Classify Research Depth

Choose the smallest depth that can responsibly answer the request:

| Depth | Use when | Typical work |
| --- | --- | --- |
| `none` | The task is local, obvious, already scoped, or better handled by `$chat` or `$work`. | Do not run deep research; return the appropriate next step. |
| `light` | A few current facts, official docs, or examples would remove uncertainty. | Quick search or official-doc check; no tool installation unless already available. |
| `standard` | The task involves technical choice, integration, architecture tradeoff, or known unknowns. | Multi-source web research, official docs, comparable examples, and a short cited brief. |
| `deep` | The problem is high-impact, fast-moving, complex, security-sensitive, market/legal/policy-sensitive, or lacks an obvious solution. | Multi-angle research, source extraction, cross-checking, and a structured research brief. |
| `exhaustive` | The user explicitly asks for comprehensive research, failure cost is high, or the answer depends on broad prior art. | Multiple research tools or agents, broad source collection, contrarian views, and explicit uncertainty mapping. |

Escalate depth when facts may have changed recently, the decision could cost significant time or money, primary sources are needed, or there is a meaningful chance that memory-only reasoning would be wrong.

Treat tool, integration, automation, CLI, MCP, plugin, API, data-pipeline, and developer-workflow requests as reuse-first by default. The minimum responsible depth is usually `standard` when the answer depends on selecting among existing projects, libraries, registries, or vendor APIs, because maintenance, license, integration cost, and real-world adoption need cross-checking.

When the user invokes `$chat-deep`, assume the problem is non-trivial enough to deserve meaningful breadth and depth. Do not satisfy reuse-first discovery with a shallow list of links. Search broadly enough to expose multiple plausible wheels or industry approaches, then analyze relevant candidates deeply enough to make the reuse decision defensible.

### 3. Propose A Research Plan

Do not propose a research plan until the problem framing has been confirmed. If the framing is still uncertain, stay in the interview phase and ask another focused round of questions.

Before installing new tools or starting substantial research, ask for explicit confirmation unless the user already gave clear permission in the same turn after confirming the problem framing.

The research plan should include:

- Research depth: `light`, `standard`, `deep`, or `exhaustive`.
- Key questions the research must answer.
- Search coverage: the broad source categories that will be searched before narrowing.
- Source targets: official docs, code repositories, papers, market reports, forums, issue trackers, competitor docs, standards bodies, vendor docs, and industry practice writeups.
- Reuse-first discovery targets when applicable: GitHub/GitLab repositories, package registries, official ecosystem directories, MCP/plugin registries, CLI catalogs, SaaS/API vendors, hosted products, standards/protocols, industry workflows, templates, examples, and comparable implementations.
- Candidate evaluation criteria when applicable: feature fit, maintenance activity, release cadence, issue/PR quality, license, security posture, dependency risk, integration effort, extension points, adoption signals, and migration or exit cost.
- Candidate tools and why each is useful.
- Expected artifact path for the markdown brief.
- Time or breadth expectation when relevant.

Confirmation phrasing should be direct:

```text
请确认是否按这个调研计划继续。确认后，$chat-deep 可以安装或配置上面列出的调研工具，并会只写调研文档，不改业务代码。
```

After confirmation, create or update `RESEARCH_STATE.md` and transition `scoped -> plan_confirmed`.

### 4. Tool Selection

Prefer existing available skills or built-in web tools first. Install or configure extra tooling only when it improves the research result.

Common tool categories:

- Search and synthesis: Tavily, Perplexity Sonar, Exa, Linkup, Valyu, OpenAI web/deep research, or equivalent current search APIs.
- Crawling and extraction: Firecrawl, Bright Data, Apify, Jina Reader, browser automation, or equivalent scraping/extraction tools.
- Developer and reuse discovery: Context7, DeepWiki, GitHub search, GitLab search, package registry search, official ecosystem directories, MCP/plugin registries, CLI catalogs, official framework docs, or repository analysis tools.
- Academic and domain research: Semantic Scholar, arXiv, Crossref, PubMed, standards bodies, government or legal sources.
- Skill/plugin discovery: skills CLI, available skill registries, plugin listings, or MCP server directories.

When selecting tools, record:

- Tool name and source.
- Installation or authentication needed.
- Why the tool is needed for this request.
- Expected output or evidence type.
- Fallback if the tool is unavailable.

Do not use stale hardcoded tool lists as final authority. Search for current official docs or registries when selecting installable tools.

Update `RESEARCH_STATE.md` and the `No-repeat ledger` after tool installation, skipped installation, failed installation, or fallback selection. Transition to `tooling_ready` only when the current research can proceed with available tools.

### 5. Conduct Research

Use primary sources whenever possible. Cross-check important claims across independent sources. Separate evidence from inference.

For `standard`, `deep`, and `exhaustive` research, cover:

- Current best practices or established solutions, including industry-standard approaches that are not open-source projects.
- Prior art from comparable projects or products.
- Reusable candidates, including direct-use, wrapper/integration, fork/adapt, reference-only, and reject decisions.
- Official constraints, APIs, compatibility notes, or deprecations.
- Risks, failure modes, tradeoffs, and rejected alternatives.
- How findings map back to the user's specific project.

For tool-like requests, conduct reuse-first discovery before proposing a bespoke design. A "wheel" can be an open-source project, commercial or hosted product, official API, standard protocol, mature vendor workflow, template, CLI, plugin, MCP server, or documented industry solution. Search across likely ecosystems and record a candidate matrix with:

- Candidate name, URL, source type, license, maturity, and maintenance signal.
- Capability fit against the user's required workflow.
- Integration path: direct use, package dependency, CLI invocation, API integration, MCP/plugin installation, fork/adapt, or reference-only.
- Risks: stale releases, license mismatch, missing feature, fragile API, vendor lock-in, operational burden, security concern, or poor project fit.
- Decision: adopt, shortlist, reject, or defer, with rationale.

Prefer recommending reuse, composition, wrapping, or forking when it satisfies the core constraints with less implementation risk than custom code. Recommend building from scratch only when existing options fail clear criteria, and record that justification in `DECISIONS.md`.

Apply this adoption gate before writing the final brief:

- If any candidate is `adopt`, `Recommended Solution` must name that candidate and explain the concrete adoption path.
- If the best candidate is `shortlist`, either promote it to an adoption path with remaining validation tasks, or explain exactly what blocks adoption.
- If the recommendation is `build custom`, `DECISIONS.md` and `RESEARCH_BRIEF.md` must include `Custom Build Justification`: which candidates were considered, why each failed the core constraints, and which parts may still be reused as references.
- Do not ask `$work` to rewrite core functionality already provided by an adopted candidate. Custom code in the handoff should be limited to glue code, adapters, configuration, tests, UX integration, or project-specific extensions unless the custom build justification explicitly allows more.

For technical implementation research, inspect the local codebase enough to avoid generic recommendations when the user is asking to land the solution in an existing project. If the user is researching a new requirement without a concrete codebase dependency, keep local inspection lightweight and focus the brief on external solution discovery, decision logic, and adoption path.

During collection, write each source to `SOURCES.md` as soon as it is evaluated. During extraction, write claims and caveats to `FINDINGS.md`. During synthesis and validation, update `DECISIONS.md` and `RESEARCH_STATE.md` before continuing.

Keep the internal research heavy and the final brief useful. `SOURCES.md`, `FINDINGS.md`, and `DECISIONS.md` may contain the detailed trail; `RESEARCH_BRIEF.md` should focus on resolving the user's uncertainty, explaining the decision logic, and showing how to reuse or adopt the chosen solution.

### 6. Write The Research Brief

Write exactly one primary markdown brief unless the user asks for more. Prefer `.codex/research/YYYY-MM-DD-topic/RESEARCH_BRIEF.md` in the workspace. If there is no workspace, use a clear local research path and report it.

Assemble the brief from the persistent state files. Do not re-run searches merely to write the final brief.

Use this structure:

```markdown
# Research Brief: [Topic]

## Question
[The user's real question or uncertainty, what they need to decide, and why the problem is non-trivial.]

## Project Context
[Relevant existing-project or workflow evidence when applicable. If this is a new requirement without meaningful repository constraints, say so briefly and focus the report on the user's need.]

## Search Coverage
[Where the research looked: open-source repositories, package registries, official APIs, standards/protocols, hosted products, industry practices, forums/issues, competitor docs, or other relevant areas. Keep this concise but enough to prove breadth.]

## Candidate Solutions
[Candidates and industry approaches considered. For relevant candidates, include source links, source type, fit, maturity or adoption signal, integration path, risks, and adopt/shortlist/reject/defer decision. Compress weak or low-relevance candidates.]

## Evaluation Logic
[The criteria and reasoning used to judge candidates. Explain why the recommendation follows from the evidence.]

## Recommendation
[The preferred direction and rationale. For tool-like requests, name the adopted project, package, API, CLI, MCP server, plugin, fork, template, standard, hosted product, or industry approach whenever one is viable.]

## Adoption Plan
[For tool-like requests: exact reuse path, install or integration method, wrapper/adaptor boundary, fork strategy when relevant, project files likely affected, and what custom code is allowed. If building custom, include Custom Build Justification.]

## Why Not The Others
[Main alternatives and why they are not the current recommendation. Keep this decision-oriented, not exhaustive.]

## Risks And Unknowns
[Known risks, uncertainty, stale-source risk, compatibility risk, or missing credentials.]

## Verification Strategy
[How $work should prove the implementation is correct.]

## Work Handoff
[A $work-ready implementation handoff following the chat/task capsule style: problem, goals, non-goals, execution mode, topology, task capsules, verification, and risks.]

## Rerun Inputs
[Query, depth, tools, source constraints, date, and any relevant versions.]
```

The brief must answer the user's uncertainty and make the recommendation logic clear. It should not include long source dumps or every weak candidate found during broad search. Cite sources for factual claims that came from the web. Keep quotes short and prefer summaries.

When the brief is ready, update `RESEARCH_STATE.md` to `handoff_ready`. After the user-facing closeout is delivered, update it to `complete`.

### 7. Produce The User-Facing Closeout

After writing the brief, respond in the user's language with:

- The research depth used.
- Tools installed or used.
- The markdown brief path.
- The recommended implementation direction.
- Any unresolved risks or missing credentials.
- A reminder that implementation requires explicit `$work`.

Do not start implementation in `$chat-deep`.

## Handoff To `$work`

The final `Work Handoff` section in the markdown brief must be sufficient for `$work` to execute without redoing research. Include:

- Problem statement.
- Current context with links to local files when relevant.
- Goals and non-goals.
- Acceptance criteria.
- Reuse decision: direct use, wrap/integrate, fork/adapt, reference-only, or build custom, with rationale when applicable.
- Adopted source: the exact project, package, API, CLI, MCP server, plugin, template, fork, or "none".
- Adoption path: install/use directly, wrap/integrate, fork/adapt, vendor/template import, API integration, reference-only, or build custom.
- Custom-code boundary: what may be written locally, and what must remain provided by the adopted source.
- Custom Build Justification, required when the handoff asks `$work` to build functionality that a candidate project appeared to provide.
- Execution mode: `simple-main`, `state-main`, or `delegated-state`.
- Plan topology: `linear` or `branched`.
- Ordered task capsules with purpose, exact work, likely files, allowed read scope, allowed write scope, non-goals, acceptance criteria, focused verification, dependencies, and rollback notes when relevant.
- Final verification expectations.
- Risks and open questions.

If the solution requires additional implementation dependencies, `$chat-deep` may recommend them in the handoff, but must not install them as part of research unless they are strictly research tooling.

## Language Rule

Mirror the user's language in all user-facing responses, research-plan prompts, closeouts, and markdown brief prose unless the user requests a different language. Keep protocol tokens such as `$chat-deep`, `$chat`, `$work`, `simple-main`, `state-main`, and `delegated-state` unchanged.
