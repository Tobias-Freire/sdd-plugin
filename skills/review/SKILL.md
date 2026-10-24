---
name: review
description: Review everything that was done in a task, looking for improvements, security flaws and optimizations, and list the findings in a table with location, reason, fix and priority. Use when the user invokes /review or asks for a review of the work.
disable-model-invocation: false
argument-hint: "[task-slug, branch, PR number or path]"
allowed-tools: Read, Grep, Glob, Bash(git diff:*), Bash(git log:*), Bash(git status:*), Bash(git show:*), Bash(git branch:*), Bash(gh pr view:*), Bash(gh pr diff:*), Write, Edit, Bash(git check-ignore:*), Bash(git rev-parse:*)
---

# /review

Review all the work done for a task and report what should be improved. This skill does not change code: it only reports findings (and updates `HANDOFF.md`).

Arguments received: `$ARGUMENTS`

Before anything else, read `HANDOFF.md` if it exists (see the Handoff section).

## 1. Define the scope

1. Determine what to review, in this order:
   - A PR number, branch or path in the arguments.
   - The commits and uncommitted changes of the current task (`git status`, `git diff`, `git diff --staged`, and `git log` against the base branch).
   - A task slug: read `.sdd/<task-slug>/spec.md`, `plan.md` and `todo.md` to compare the result with what was specified.
2. If the scope is unclear, state your assumption and continue. Ask only if it changes the review materially.
3. Read the changed files in full, not only the diff, plus the code that calls or is called by them.

## 2. Review

Look at every angle that applies:

- **Correctness**: logic errors, unhandled edge cases, wrong assumptions, race conditions, resource leaks.
- **Spec conformance**: requirements or acceptance criteria that are missing or only partly met.
- **Security**: injection, broken authentication or authorization, exposed secrets, unsafe deserialization, unvalidated input, insecure defaults, vulnerable dependencies, sensitive data in logs.
- **Performance and optimization**: needless work, N+1 queries, blocking calls, missing indexes or caching, memory use, bundle size.
- **Code quality**: clarity, naming, function size, duplication, dead code, magic values, coupling, consistency with the surrounding code.
- **Tests**: missing unit tests for methods, untested edge cases, brittle or meaningless tests, weak assertions.
- **Error handling and observability**: swallowed errors, unclear messages, missing logs where they matter.
- **Maintainability and docs**: missing or outdated documentation, hard-to-change design, breaking changes.

Only report real issues you verified in the code. Do not pad the list with style preferences, and do not report what linters already enforce unless it is being violated.

## 3. Report: findings table first

Start the report with a table. Use one row per finding, ordered from the most to the least important:

| # | Location | Why it needs a fix | Suggested fix | Priority |
|---|----------|--------------------|---------------|----------|
| 1 | `path/file.ext:42` | The problem and its impact. | The concrete change to make. | Urgent |

Rules for the table:

- **Location**: file and line (or range), or the component when it spans files.
- **Why it needs a fix**: the impact in one or two sentences.
- **Suggested fix**: specific enough to act on without further research.
- **Priority**: exactly one of **Low**, **Medium**, **High** or **Urgent**.
  - **Urgent**: exploitable security flaw, data loss or corruption, or a failure that breaks the main flow. Fix before anything else.
  - **High**: bug or risk likely to affect users, or a requirement not met.
  - **Medium**: worthwhile improvement to quality, performance or tests with limited immediate risk.
  - **Low**: minor polish, naming, small optimizations.

If nothing was found, say so plainly and mention what you checked.

## Handoff

`HANDOFF.md` lives at the root of the git repository where the task started. It carries context between skills, so the next step does not have to rediscover it. It is never committed.

- **Read it first**, if it exists. Use it as context and verify anything critical against the files.
- **Keep it out of git**: if `git check-ignore -q HANDOFF.md` fails, add `HANDOFF.md` to the file printed by `git rev-parse --git-path info/exclude`. Never stage or commit it, not even when everything pending is committed.
- **Write it last**: create it if missing, otherwise update the section `## <task-slug>` (one section per task; leave other tasks untouched). Keep these parts: stage (this skill and date), done, decisions, artifacts (paths, issue and PR numbers, branch) and next steps with open questions. Replace outdated content instead of appending, keep the section under about 60 lines and never write secrets.
- **For this skill**: Record the findings table (or a condensed version with location, priority and status), and, once the user chooses, which ones will be fixed and which were fixed (update the entry when that happens). Next step: apply the chosen fixes, then `/pr`.

## 4. After the table

Add a short summary with: the number of findings per priority, what was done well, anything you could not verify (and why), and a suggested order for the fixes.

Then ask the user which findings to fix. Do not apply any fix until the user chooses. Fixes can then be applied with `/build` or directly.
