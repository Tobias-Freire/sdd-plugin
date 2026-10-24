---
name: issue
description: Create a GitHub issue in the task's repository with appropriate labels and a detailed description. Use when the user invokes /issue or asks to open an issue for a task, bug or feature.
disable-model-invocation: false
argument-hint: "<task description or task-slug>"
allowed-tools: Read, Grep, Glob, Bash(gh issue create:*), Bash(gh issue view:*), Bash(gh issue list:*), Bash(gh label list:*), Bash(gh repo view:*), Bash(gh auth status:*), Bash(git remote:*), Bash(git status:*), Bash(git log:*)
---

# /issue

Create a well-written GitHub issue for a task in the repository the task belongs to.

Task received: `$ARGUMENTS`

## 1. Gather the content

1. Use the arguments as the task. If they name a task slug, or the task has a `.sdd/<task-slug>/spec.md`, build the issue from the spec. With no arguments, use the task discussed in the conversation. If there is none, ask the user to describe it.
2. Read the relevant code and docs when the description needs concrete references (files, components, error messages).
3. Check the repository: `gh repo view --json nameWithOwner,url` and `git remote -v`. If `gh` is not installed or authenticated, tell the user (they can run `! gh auth login`) and stop. If the directory is not a GitHub repository, stop and say so.

## 2. Avoid duplicates

Run `gh issue list --state open --search "<keywords>"`. If a similar issue exists, show it to the user and ask whether to continue or reuse it.

## 3. Choose labels

1. List the labels that exist with `gh label list`.
2. Pick the appropriate ones from the existing labels only, such as `bug`, `enhancement`, `documentation`, `security`, `performance`, `good first issue`, or the repository's own naming. Typically one type label plus any relevant area or priority label.
3. Never create new labels without asking. If no existing label fits, create the issue without it and mention this.

## 4. Write the issue

**Title**: short, specific and in the imperative or descriptive form, under about 70 characters. No type prefixes unless the repository uses them.

**Body**, adapted to the kind of task (drop sections that do not apply):

```markdown
## Summary
What needs to be done and why, in a few sentences.

## Context
Background, motivation and links to related code, issues, PRs or docs.

## Details
For features: the expected behavior, requirements and technical approach.
For bugs: steps to reproduce, expected vs actual behavior, environment, logs.

## Acceptance criteria
- [ ] Concrete, verifiable items.

## Out of scope
What this issue does not cover.

## Notes
Open questions, risks, suggested approach or dependencies.
```

When a spec exists, reuse its requirements and acceptance criteria instead of rewriting them differently. Write in English unless the repository or the user uses another language.

## 5. Create it

1. Show the user the title, labels and body, and ask for confirmation. Skip the confirmation only if the user already approved this exact content.
2. Create it, passing the body through a file or heredoc to preserve formatting:

   ```bash
   gh issue create --title "<title>" --body-file <file> --label "<label1>,<label2>"
   ```

   Add `--assignee` or `--milestone` only if the user asked for them.
3. Do not include secrets, tokens or private data in the issue.

## 6. Report

Reply with the issue number and URL, the labels applied and any labels you could not apply. Remember the issue number so `/pr` can link it, and suggest including it in the branch name or commit when useful.
