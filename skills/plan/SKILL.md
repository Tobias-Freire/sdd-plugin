---
name: plan
description: Break a specification into independent, executable parts with checks and tests for each part, producing plan.md and todo.md. Use when the user invokes /plan or asks to plan the implementation of a spec.
disable-model-invocation: false
argument-hint: "[task-slug or path to spec.md]"
allowed-tools: Read, Grep, Glob, Write, Edit, Bash(git log:*), Bash(git status:*), Bash(ls:*), Bash(git check-ignore:*), Bash(git rev-parse:*)
---

# /plan

Break the specification into parts that can be built and verified one at a time. Produce two files: `plan.md` (the parts) and `todo.md` (the checklist of checks for each part).

Arguments received: `$ARGUMENTS`

Before anything else, read `HANDOFF.md` if it exists (see the Handoff section).

## 1. Locate the spec

1. Use the argument as a task slug or as a path to `spec.md`. With no argument, use the most recently modified `.sdd/*/spec.md`.
2. If no spec exists, tell the user and suggest `/spec`. Do not invent one.
3. Read the spec fully, then explore the code it touches so the plan matches the real structure.

Plan files go in the same directory as `spec.md`.

## 2. Split into parts

Each part must be:

- **Independent**: it can be built and verified without the later parts. Order parts so each one only depends on earlier parts.
- **Executable**: small enough to finish in one focused step, large enough to deliver something observable.
- **Verifiable**: it has its own checks and tests that prove it works.
- **Traceable**: it lists the spec requirements (`R1`, `R2`, ...) it covers. Every requirement and acceptance criterion must be covered by at least one part.

Put groundwork first (setup, data model, shared utilities), then features, then integration and polish.

## 3. Write `plan.md`

Start with a short summary and a table of the parts, then one section per part:

```markdown
## Part N: <name>

- **Goal**: what this part delivers.
- **Covers**: R1, R3.
- **Depends on**: Part M, or none.
- **Files and components**: what will be created or changed.
- **Approach**: the key steps and decisions.
- **Tests**: unit and integration tests to write (name the behavior each one proves).
- **Verification**: commands or manual checks that prove the part is done.
- **Risks**: what could go wrong and how to detect it.
```

If the task involves code, every part with logic must list unit tests for its methods.

## 4. Write `todo.md`

A Markdown checklist grouped by part. Each part gets its implementation steps and its checks, so `/build` can tick them off:

```markdown
## Part N: <name>

- [ ] Implement <step>
- [ ] Unit test: <behavior>
- [ ] Check: <command or observable result>
```

Checks must be concrete: an exact command, an expected output or an observable behavior. Never write checks such as "works correctly".

## Handoff

`HANDOFF.md` lives at the root of the git repository where the task started. It carries context between skills, so the next step does not have to rediscover it. It is never committed.

- **Read it first**, if it exists. Use it as context and verify anything critical against the files.
- **Keep it out of git**: if `git check-ignore -q HANDOFF.md` fails, add `HANDOFF.md` to the file printed by `git rev-parse --git-path info/exclude`. Never stage or commit it, not even when everything pending is committed.
- **Write it last**: create it if missing, otherwise update the section `## <task-slug>` (one section per task; leave other tasks untouched). Keep these parts: stage (this skill and date), done, decisions, artifacts (paths, issue and PR numbers, branch) and next steps with open questions. Replace outdated content instead of appending, keep the section under about 60 lines and never write secrets.
- **For this skill**: Record the paths of `plan.md` and `todo.md`, the list of parts with their one-line goals and the main risks. Next step: `/build`.

## 5. Final report

Reply briefly with the paths of `plan.md` and `todo.md`, the number of parts and any risks or open questions. Suggest `/build` as the next step. Do not start implementing.
