---
name: build
description: Build a task part by part from spec.md, plan.md and todo.md, waiting for user validation after each part. Supports --validate and --all. Use when the user invokes /build or asks to implement a planned task.
disable-model-invocation: false
argument-hint: "[--validate] [--all] [task-slug]"
---

# /build

Implement the task described by `spec.md`, `plan.md` and `todo.md`, one part at a time.

Arguments received: `$ARGUMENTS`

Before anything else, read `HANDOFF.md` if it exists (see the Handoff section).

## 1. Parse the arguments

| Flag | Effect |
|------|--------|
| *(none)* | Build the next unfinished part, then stop and wait for the user's validation. |
| `--validate` | Do not build. Run the checks and tests of the current part (the last built part that is not yet validated) and report the result. |
| `--all` | Build every remaining part and run all tests and checks, without waiting for validation between parts. |

Any other text is a task slug or a free-form instruction. Follow it when it does not conflict with the rules below.

## 2. Load the context

1. Find the task directory in `.sdd/<task-slug>/`. With no slug, use the most recently modified `.sdd/*/`.
2. Read `spec.md`, `plan.md` and `todo.md`. If any is missing, stop and tell the user which skill produces it (`/spec` or `/plan`).
3. In `todo.md`, parts whose items are all checked are done. The next part is the first one with unchecked items. If all are checked, report that the task is complete.
4. Check `git status`. If there are unrelated pending changes, leave them alone.

## 3. Build a part

1. Re-read the part in `plan.md`, the requirements it covers in `spec.md` and the existing code around it. Follow the conventions already in the repository.
2. Implement only this part. Do not start the next part and do not change the plan silently. If the plan is wrong or incomplete, stop, explain and ask the user.
3. Write the tests listed for the part together with the code.
4. Run the part's checks and tests. Fix failures within the part's scope. Never weaken, skip or delete a test to make it pass.
5. Tick the completed items in `todo.md`. Leave failing or unverified items unchecked and note why.

## 4. Code quality rules

When the task involves code:

- Add at least **unit tests for every method or function** that contains logic, covering the normal case, edge cases and error cases.
- Follow clean code principles: clear and intention-revealing names, small functions with a single responsibility, no duplication, no dead code, no magic values, shallow nesting.
- Keep the code readable and easy to understand. Prefer simple solutions. Add comments only to explain why, not what.
- Match the style, structure and idioms of the surrounding code.
- Handle errors explicitly and never swallow them silently.
- Do not add features, abstractions or dependencies beyond what the spec requires.

## 5. Mode behavior

### Default

After building one part and running its checks, stop and report:

- The part built and the files changed.
- The tests and checks that ran, with the results.
- What the user should look at to validate it.

Then wait. Do not continue until the user validates the part, for example by running `/build` again. If the user asks for changes, apply them to the same part first.

### `--validate`

Run every check and test listed in `todo.md` for the current part, plus the full test suite of the project when it exists. Report each result. Tick the items that pass and keep the others unchecked. Do not change code in this mode, except to fix a trivial problem the user explicitly asks you to fix.

### `--all`

Build the remaining parts in order without stopping between them. After each part, run its checks and tests before moving on. After the last part, run all checks and the full test suite. Stop early only when a part fails and cannot be fixed within its scope, when the plan is wrong or when a decision is needed from the user. Report what is done and what is blocked.

## 6. Safety rules

- Do not commit or push unless the user asks. Suggest `/commit` when a part is validated.
- Do not run destructive commands (dropping data, `rm -rf`, force operations) without explicit confirmation.
- Never write secrets or credentials into code or logs.

## Handoff

`HANDOFF.md` lives at the root of the git repository where the task started. It carries context between skills, so the next step does not have to rediscover it. It is never committed.

- **Read it first**, if it exists. Use it as context and verify anything critical against the files.
- **Keep it out of git**: if `git check-ignore -q HANDOFF.md` fails, add `HANDOFF.md` to the file printed by `git rev-parse --git-path info/exclude`. Never stage or commit it, not even when everything pending is committed.
- **Write it last**: create it if missing, otherwise update the section `## <task-slug>` (one section per task; leave other tasks untouched). Keep these parts: stage (this skill and date), done, decisions, artifacts (paths, issue and PR numbers, branch) and next steps with open questions. Replace outdated content instead of appending, keep the section under about 60 lines and never write secrets.
- **For this skill**: Record which parts are built and validated, the files changed, test results, decisions that deviate from the plan and what is blocked. Next step: `/build` for the next part, or `/commit` once a part is validated.

## 7. Final report

Reply briefly with the parts built, the test and check results, the files left unchecked in `todo.md` and the next step.
