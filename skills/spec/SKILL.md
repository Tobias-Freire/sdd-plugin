---
name: spec
description: Generate a detailed specification (spec.md) for a task before any code is written. Use when the user invokes /spec or asks to specify, scope or document a task or feature.
disable-model-invocation: false
argument-hint: "<task description>"
allowed-tools: Read, Grep, Glob, Write, Bash(git log:*), Bash(git status:*), Bash(ls:*), Edit, Bash(git check-ignore:*), Bash(git rev-parse:*)
---

# /spec

Turn a task description into a detailed, unambiguous specification saved as `spec.md`. The spec says **what** must be built and **why**, not how to break the work down (that is `/plan`).

Task received: `$ARGUMENTS`

Before anything else, read `HANDOFF.md` if it exists (see the Handoff section).

## 1. Understand the task

1. If the arguments are empty, use the task discussed in the conversation. If there is none, ask the user to describe it.
2. Explore the repository (structure, conventions, related code, existing docs) so the spec fits the project instead of being generic.
3. Ask the user clarifying questions only for decisions you cannot resolve from the request or the code. Prefer sensible defaults and record them under **Assumptions**.

## 2. Choose the output location

Save artifacts of the SDD workflow in `.sdd/<task-slug>/`, where `<task-slug>` is a short kebab-case name for the task. Create the directory if needed. If the user names another location, use it. Reuse the same directory for `plan.md`, `todo.md` and `test-script.md`.

If `spec.md` already exists there, read it and update it instead of overwriting it blindly.

## 3. Write `spec.md`

Use these sections, dropping one only when it truly does not apply:

1. **Title and summary**: one paragraph describing the task.
2. **Context and motivation**: the problem, who is affected and why it matters now.
3. **Goals**: what must be true when the task is done.
4. **Non-goals**: what is explicitly out of scope.
5. **Requirements**: numbered, testable statements (`R1`, `R2`, ...). Use "must" for mandatory and "should" for optional. Each requirement must be verifiable.
6. **User flows or behavior**: step-by-step descriptions of the main flow and the alternative and error flows.
7. **Edge cases and error handling**: boundary values, empty or invalid input, failures of dependencies, concurrency.
8. **Technical approach**: affected components and files, data model or API changes, key design decisions and the alternatives considered.
9. **Non-functional requirements**: security, performance, accessibility, compatibility, observability, when relevant.
10. **Acceptance criteria**: a checklist that decides whether the task is done, each item traceable to a requirement.
11. **Assumptions and open questions**: defaults you chose and points still needing an answer.

Writing rules:

- Be specific. Replace vague words ("fast", "user-friendly", "robust") with measurable statements.
- Do not invent facts about the codebase. Read the code or mark the point as an open question.
- Do not write implementation code. Short signatures or schemas are fine when they remove ambiguity.
- Write in English unless the user asks otherwise.

## Handoff

`HANDOFF.md` lives at the root of the git repository where the task started. It carries context between skills, so the next step does not have to rediscover it. It is never committed.

- **Read it first**, if it exists. Use it as context and verify anything critical against the files.
- **Keep it out of git**: if `git check-ignore -q HANDOFF.md` fails, add `HANDOFF.md` to the file printed by `git rev-parse --git-path info/exclude`. Never stage or commit it, not even when everything pending is committed.
- **Write it last**: create it if missing, otherwise update the section `## <task-slug>` (one section per task; leave other tasks untouched). Keep these parts: stage (this skill and date), done, decisions, artifacts (paths, issue and PR numbers, branch) and next steps with open questions. Replace outdated content instead of appending, keep the section under about 60 lines and never write secrets.
- **For this skill**: Record the path of `spec.md`, a short summary of the scope, the assumptions and the open questions. Next step: `/plan`.

## 4. Final report

Reply briefly with the path of `spec.md`, the main assumptions and the open questions, then suggest `/plan` as the next step. Do not start implementing.
