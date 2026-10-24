---
name: pr
description: Generate the title and description of a pull request, ask the user to approve them, then open the PR on GitHub and link the related issue if there is one. Use when the user invokes /pr or asks to open a pull request.
disable-model-invocation: false
argument-hint: "[base-branch] [issue number]"
allowed-tools: Read, Grep, Glob, Bash(gh pr create:*), Bash(gh pr view:*), Bash(gh pr list:*), Bash(gh issue view:*), Bash(gh repo view:*), Bash(gh auth status:*), Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git branch:*), Bash(git remote:*), Bash(git rev-parse:*), Bash(git push:*), Write, Edit, Bash(git check-ignore:*)
---

# /pr

Write a pull request title and description from the work on the current branch, get the user's approval, then open the PR on GitHub.

Arguments received: `$ARGUMENTS`

Before anything else, read `HANDOFF.md` if it exists (see the Handoff section).

## 1. Check the state

1. Run `git status`, `git branch --show-current` and `gh repo view --json nameWithOwner,defaultBranchRef`. If `gh` is missing or not authenticated, tell the user (they can run `! gh auth login`) and stop.
2. The base branch is the one given in the arguments, otherwise the repository's default branch.
3. If the current branch is the base branch, stop and ask the user to move the work to a feature branch. Do not create branches or move commits without asking.
4. If there are uncommitted changes, tell the user and suggest `/commit` first. Continue only if they confirm.
5. If a PR already exists for the branch (`gh pr list --head <branch>`), show it and ask whether to update it instead of creating another.

## 2. Collect the content

1. Read the commits and the full diff against the base: `git log <base>..HEAD` and `git diff <base>...HEAD`.
2. Read the task artifacts when they exist in `.sdd/<task-slug>/` (`spec.md`, `plan.md`, `test-script.md`, test report) to describe the intent, the requirements met and the tests performed.
3. Find the related issue, in this order: an issue number in the arguments, one mentioned in the conversation or in `.sdd/<task-slug>/`, one referenced in the branch name or commits (`#123`). Confirm it with `gh issue view <number>`. If none is found, there is no related issue; do not guess.

## 3. Write the title and description

**Title**: under about 70 characters, imperative mood, describing what the PR does. Follow the repository's convention (for example Conventional Commits, as in `git log`) when there is one.

**Description**:

```markdown
## Summary
What this PR does and why, in a few sentences.

## Changes
- Main changes, grouped by area, as a short bullet list.

## How to test
Exact steps or commands to verify the change, and the tests added or run with their results.

## Notes
Breaking changes, migrations, follow-ups, screenshots, or points the reviewer should look at closely.

Closes #<issue>
```

Rules:

- Describe the intent and the impact, not a line-by-line listing of the diff.
- Include `Closes #<issue>` only when a related issue was identified. Use `Refs #<issue>` if the PR only partly addresses it. This links the PR to the issue and closes it on merge.
- Do not claim tests were run unless they were. Do not include secrets.
- Write in English unless the repository or the user uses another language.

## 4. Ask for approval

Show the user the base branch, head branch, title, description and the linked issue. Ask whether to open the PR as it is, as a draft, or with changes. **Do not open the PR before the user approves.** If the user asks for changes, revise and ask again.

## 5. Open the PR

After approval:

1. Make sure the branch is on the remote. If it is not pushed or is behind, tell the user and push it with `git push -u origin <branch>` (never force-push).
2. Create the PR, passing the body from a file to preserve formatting:

   ```bash
   gh pr create --base <base> --head <branch> --title "<title>" --body-file <file>
   ```

   Add `--draft` if the user chose a draft. Add `--label`, `--reviewer` or `--assignee` only if the user asked for them.
3. If an issue is related, check that it is linked (`gh pr view --json closingIssuesReferences`). If the keyword did not link it, add a comment on the issue pointing to the PR.

## Handoff

`HANDOFF.md` lives at the root of the git repository where the task started. It carries context between skills, so the next step does not have to rediscover it. It is never committed.

- **Read it first**, if it exists. Use it as context and verify anything critical against the files.
- **Keep it out of git**: if `git check-ignore -q HANDOFF.md` fails, add `HANDOFF.md` to the file printed by `git rev-parse --git-path info/exclude`. Never stage or commit it, not even when everything pending is committed.
- **Write it last**: create it if missing, otherwise update the section `## <task-slug>` (one section per task; leave other tasks untouched). Keep these parts: stage (this skill and date), done, decisions, artifacts (paths, issue and PR numbers, branch) and next steps with open questions. Replace outdated content instead of appending, keep the section under about 60 lines and never write secrets.
- **For this skill**: Record the PR number, URL, base and head branches and the linked issue. Next step: `/generate-tests`.

## 6. Report

Reply with the PR number and URL, the base and head branches and the issue it is linked to, if any.
