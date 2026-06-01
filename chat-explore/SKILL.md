---
name: chat-explore
description: Read-only divergent-thinking skill surfaced as /explore. Use when the user asks /explore to generate five distinct solution approaches, estimate the likelihood of generating each approach with a 1-10 score, and label each idea as 常见, 少见, or 极度罕见.
---

# Chat Explore

## Overview

Use this skill when the user invokes `/explore` or asks for deliberately divergent solution generation before choosing a path. The skill produces five meaningfully different approaches, estimates how likely the agent would be to generate each approach, and labels every approach by rarity.

This skill belongs to the chat skill family. It is for thinking, comparison, and option discovery only. It must not implement, edit files, install dependencies, stage, commit, push, or open pull requests.

## Operating Contract

- Treat `/explore` as a read-only conversation mode.
- Do not create, modify, delete, rename, format, or generate workspace files while this skill is active.
- Do not use `apply_patch`, shell redirection, package managers, scaffolding commands, migrations, formatters, or code generation that writes to disk.
- Do not install dependencies, launch implementation agents, stage files, commit, push, or open pull requests.
- Use read-only project inspection only when the user question depends on repository context.
- If repository inspection is not needed, answer directly from the prompt and stated constraints.
- Separate evidence from inference when drawing conclusions from code or documents.
- If a proposed option would require later implementation, describe it as a possible path, not as work already approved.
- If the user asks to implement one of the options, stop `/explore` and require a separate execution request or the appropriate implementation workflow.

## Applicability Gate

Use this skill when the user wants:

- five sharply different ways to solve a product, design, architecture, process, or implementation problem
- a spread that includes conventional and unconventional thinking
- explicit likelihood scoring for why the agent would or would not naturally produce each idea
- rarity labels using `常见`, `少见`, and `极度罕见`
- a decision-oriented comparison before implementation

Do not use this skill for:

- already-scoped implementation tasks
- code review, security review, or build-fix work
- tasks where the user wants a single recommended answer without divergent exploration
- requests that require workspace edits during the same turn

## Workflow

### 1. Frame The Question

Restate the problem in one or two sentences. Include the known constraints, target user, project context, or success criteria when they are available.

If the prompt is too ambiguous to produce useful options, ask at most three focused questions. If the ambiguity is tolerable, state the assumptions and proceed.

### 2. Generate Five Distinct Approaches

Create exactly five approaches. Make them meaningfully different in mechanism, risk profile, implementation shape, user experience, or strategic intent.

The set should usually include:

- one conventional low-risk approach
- one balanced approach that is likely to be practical
- one architecture or systems-oriented approach when relevant
- one product or workflow-oriented approach when relevant
- one deliberately unusual approach that may reveal a hidden opportunity

Do not pad the list with minor variations. If two approaches would touch the same core lever, merge or replace one of them.

### 3. Score Generation Likelihood

For each approach, assign a `生成概率评分` from 1 to 10.

This score means: "How likely is this agent, given the prompt and context, to generate this approach without special prompting?"

It is not a statistical probability, market forecast, implementation success probability, or business value score.

Use the scale this way:

- `1-2`: extremely unlikely to be generated unless the user explicitly asks for strange or lateral ideas
- `3-4`: unlikely, but plausible after broad exploration
- `5-6`: moderately likely, especially for an experienced agent
- `7-8`: likely and close to standard professional advice
- `9-10`: very likely, obvious, or strongly implied by the prompt

### 4. Label Rarity

Assign one rarity label to each approach:

- `常见`: likely to appear in ordinary professional brainstorming; usually maps to scores `7-10`
- `少见`: less obvious or more context-dependent; usually maps to scores `4-6`
- `极度罕见`: highly lateral, counterintuitive, or niche; usually maps to scores `1-3`

The label may deviate from the score band only when there is a clear reason. If it does, explain the reason in the rationale.

### 5. Compare And Recommend

After listing the five approaches, identify:

- the safest default
- the highest-upside option
- the option most worth discussing further
- the next validation step that would best distinguish between the strongest options

Keep recommendations advisory. Do not convert the exploration into an implementation plan unless the user explicitly asks for that next step after `/explore`.

## Output Format

Use this structure:

```text
我对问题的理解：
...

5 种截然不同的方案：

1. <方案名>
   核心思路：...
   生成概率评分：<1-10>/10
   稀有度：常见 | 少见 | 极度罕见
   适合场景：...
   主要代价：...
   主要风险：...
   下一步验证：...
   为什么会/不会自然想到：...

2. ...

快速判断：
- 最稳妥：...
- 最高上限：...
- 最值得继续追问：...
- 最能区分方案优劣的下一步验证：...

只读边界：
- 本次只是发散和比较，没有修改文件、安装依赖、提交或推送。
```

When the user asks in English, translate the headings naturally while preserving the same fields and the three rarity labels unless they ask for localized labels.

## Quality Bar

- The five approaches must be genuinely distinct.
- Each score must have a reason tied to the prompt, project context, or common agent behavior.
- Rare ideas should still be useful enough to evaluate, not random novelty.
- Do not hide uncertainty. State assumptions when context is thin.
- Prefer concise, decision-ready analysis over long generic brainstorming.
