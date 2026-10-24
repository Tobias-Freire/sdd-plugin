---
name: generate-tests
description: Generate a manual test script (test-script.md) covering every scenario, including edge cases, of a feature or task. Use when the user invokes /generate-tests or asks for a manual test plan or QA script.
disable-model-invocation: false
argument-hint: "[feature description or task-slug]"
allowed-tools: Read, Grep, Glob, Write, Edit, Bash(git log:*), Bash(git diff:*), Bash(git status:*), Bash(ls:*), Bash(git check-ignore:*), Bash(git rev-parse:*)
---

# /generate-tests

Produce a manual test script that covers all scenarios of a feature or task, including edge cases. The script must be precise enough for a person, or for `/execute-tests`, to run it step by step without guessing.

Arguments received: `$ARGUMENTS`

Before anything else, read `HANDOFF.md` if it exists (see the Handoff section).

## 1. Gather the source of truth

1. If the arguments name a task slug, read `.sdd/<task-slug>/spec.md` (and `plan.md` if present). With no arguments, use the most recently modified `.sdd/*/` or, failing that, the feature discussed in the conversation or in the current `git diff`.
2. Read the code that implements the feature to learn the real routes, screens, commands, inputs and validations. Do not test what you only imagine.
3. Ask the user only for what you cannot find: base URL, test accounts, environments, seed data.

## 2. Identify the scenarios

Cover every category that applies:

- **Happy path**: each main flow from start to finish.
- **Alternative flows**: optional steps, different roles or permissions, different configurations.
- **Validation and invalid input**: empty, missing, too long, wrong type, malformed, special characters, injection strings, unicode.
- **Boundary values**: minimum, maximum, just inside and just outside each limit; zero, one, many.
- **Error handling**: dependency failures, timeouts, network loss, unavailable services, unexpected responses.
- **State and data**: first use with no data, existing data, duplicates, concurrent actions, repeated submissions, stale sessions.
- **Security**: unauthenticated and unauthorized access, access to other users' data, exposure of sensitive data.
- **Non-functional**: responsive layouts, accessibility basics, performance with larger data, compatibility, when relevant.
- **Regression**: neighbouring behavior that the change could break.

Map each requirement and acceptance criterion of the spec to at least one test case, and say which one.

## 3. Write `test-script.md`

Save it next to the spec in `.sdd/<task-slug>/`. Use this structure:

```markdown
# Test script: <feature>

## Prerequisites
Environment, URLs, accounts, seed data and tools needed.

## Coverage matrix
| Requirement | Test cases |
|-------------|-----------|
| R1 | TC-001, TC-004 |

## Test cases

### TC-001: <short title>
- **Category**: happy path | edge case | validation | error | security | ...
- **Covers**: R1
- **Preconditions**: the state required before the test.
- **Steps**:
  1. Exact action (URL, button, command or request with payload).
  2. ...
- **Expected result**: exact, observable outcome (status code, message text, UI state, stored data).
- **Result**: [ ] Pass  [ ] Fail
- **Notes**:
```

Rules:

- One behavior per test case. Number cases sequentially (`TC-001`, `TC-002`, ...).
- Write concrete values (inputs, URLs, payloads), not "enter a valid value".
- Write expected results that can be checked without judgment calls.
- Keep steps executable by tools: prefer actions that map to browser interactions or HTTP requests.
- Group cases by flow or area, with the happy path first.
- Write in English unless the user asks otherwise.

## Handoff

`HANDOFF.md` lives at the root of the git repository where the task started. It carries context between skills, so the next step does not have to rediscover it. It is never committed.

- **Read it first**, if it exists. Use it as context and verify anything critical against the files.
- **Keep it out of git**: if `git check-ignore -q HANDOFF.md` fails, add `HANDOFF.md` to the file printed by `git rev-parse --git-path info/exclude`. Never stage or commit it, not even when everything pending is committed.
- **Write it last**: create it if missing, otherwise update the section `## <task-slug>` (one section per task; leave other tasks untouched). Keep these parts: stage (this skill and date), done, decisions, artifacts (paths, issue and PR numbers, branch) and next steps with open questions. Replace outdated content instead of appending, keep the section under about 60 lines and never write secrets.
- **For this skill**: Record the path of `test-script.md`, the number of test cases per category and the scenarios left uncovered. Next step: `/execute-tests`.

## 4. Final report

Reply briefly with the path of `test-script.md`, the number of test cases per category and any scenario you could not cover and why. Suggest `/execute-tests` as the next step. Do not run the tests.
