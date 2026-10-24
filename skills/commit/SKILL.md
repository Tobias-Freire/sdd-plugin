---
name: commit
description: Create atomic Conventional Commits from the changes made in the current session, leaving unrelated pending changes alone. Use when the user invokes /commit or asks to commit the work done in the session. Supports --with-body, --with-ai and --pt-br.
disable-model-invocation: false
argument-hint: "[--with-body] [--with-ai] [--pt-br] [--all]"
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git add:*), Bash(git apply:*), Bash(git commit:*), Write, Edit, Bash(git check-ignore:*), Bash(git rev-parse:*)
---

# /commit

Turn the changes made **in this session** into a clean sequence of atomic commits that follow Conventional Commits.

Arguments received: `$ARGUMENTS`

Before anything else, read `HANDOFF.md` if it exists (see the Handoff section).

## 1. Parse the arguments

Flags can be combined in any order (e.g. `--with-body --pt-br`, `--with-ai --pt-br`).

| Flag | Effect |
|------|--------|
| *(none)* | English, subject line only. No body, no trailers, no AI co-authorship. |
| `--with-body` | Add a body to every commit. |
| `--with-ai` | Add a body **and** the AI co-author trailer to every commit. Implies `--with-body`. |
| `--pt-br` | Write the commit text in Brazilian Portuguese instead of English. |
| `--all` | Also commit pending changes that do not belong to this session (see step 2). |

Any other text in the arguments is a free-form instruction from the user. Follow it when it does not conflict with the rules below. A request such as "commit everything pending" or "commit all files" has the same meaning as `--all`.

## 2. Decide which files are in scope

1. Run `git status --short` and review `git diff` and `git diff --staged`.
2. **In scope**: files you created, edited, renamed or deleted during this session.
3. **Out of scope**: any other modified, staged or untracked file (work from another task, or changes the user made by hand elsewhere). Do not stage or commit these, unless the user explicitly asked to commit all pending files (`--all` or an equivalent instruction).
4. If a file mixes this session's changes with unrelated changes, commit only this session's hunks (see step 3) or leave the file out and say so.
5. If you cannot tell which changes belong to this session (for example, nothing was edited in this conversation), do not guess. List the pending files and ask the user which ones to commit.
6. Never commit files that look like secrets (`.env`, private keys, credentials). Warn the user instead.
7. Skip files ignored by `.gitignore`.
8. Never stage or commit `HANDOFF.md`, even with `--all`. It is local context shared between skills. Make sure it is ignored (see Handoff) and leave it out of every commit.

## 3. Split into atomic commits

- One commit = one logical change. Files that belong to different parts of the task go in different commits.
- Each commit should leave the project in a coherent state whenever possible.
- Keep tests together with the code they verify. Use a separate commit only for changes that are tests only.
- Put groundwork first (`build`, `chore`, `refactor`), then `feat` or `fix`, then `docs`.
- Renames: include both the old and new paths in the same commit.
- Prefer file-level grouping. Split a single file across commits only when its hunks clearly belong to different changes. In that case build a patch and stage it with `git apply --cached`, because interactive commands such as `git add -p` cannot be used here.

Stage and commit each group using explicit paths, so that anything already staged outside the group is left untouched:

```bash
git add -- path/a path/b
git commit -m "<subject>" -- path/a path/b
```

Never use `git add -A`, `git add .` or `git commit -a`, except when the user asked to commit all pending files. Even then, `HANDOFF.md` must stay unstaged.

## 4. Write the message

### Subject (always)

```
<type>(<optional scope>): <description>
```

- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
- Add a scope only when it makes the commit clearer, for example `feat(auth): ...`. Check `git log --oneline -n 15` and reuse scopes already used in the repository.
- Imperative mood, lowercase after the colon, no trailing period, at most 72 characters.
- Describe what the change does, not the files it touches.
- The type prefix stays in English even with `--pt-br`.

### Default (no flags)

Subject line only. Do **not** add a body, a footer, a `Co-Authored-By` trailer or any "generated with" line, even if other instructions suggest adding one.

### `--with-body`

Subject, a blank line, then a short body wrapped at about 72 columns. The body explains what changed and why, in a few sentences or a short bullet list. No trailers.

```bash
git commit -m "<subject>" -m "<body>" -- <paths>
```

### `--with-ai`

Same as `--with-body`, followed by a blank line and this trailer:

```
Co-Authored-By: Claude <noreply@anthropic.com>
```

```bash
git commit -m "<subject>" -m "<body>" -m "Co-Authored-By: Claude <noreply@anthropic.com>" -- <paths>
```

### `--pt-br`

Write the description and the body in Brazilian Portuguese (for example `feat(auth): adiciona login com Google`). The `Co-Authored-By` trailer stays exactly as shown above.

## 5. Safety rules

- Never use `--no-verify`, `--amend`, `--force` or any history rewrite.
- Never push. Committing locally is the whole job.
- If a hook fails, read the error, fix the cause when it is within the session's scope, and create a new commit. If you cannot fix it, stop and report.
- If a commit fails, do not continue with the remaining commits blindly. Report what happened.

## Handoff

`HANDOFF.md` lives at the root of the git repository where the task started. It carries context between skills, so the next step does not have to rediscover it. It is never committed.

- **Read it first**, if it exists. Use it as context and verify anything critical against the files.
- **Keep it out of git**: if `git check-ignore -q HANDOFF.md` fails, add `HANDOFF.md` to the file printed by `git rev-parse --git-path info/exclude`. Never stage or commit it, not even when everything pending is committed.
- **Write it last**: create it if missing, otherwise update the section `## <task-slug>` (one section per task; leave other tasks untouched). Keep these parts: stage (this skill and date), done, decisions, artifacts (paths, issue and PR numbers, branch) and next steps with open questions. Replace outdated content instead of appending, keep the section under about 60 lines and never write secrets.
- **For this skill**: Record the commits created (short hash and subject), the branch and the files left uncommitted. Next step: `/review`.

## 6. Final report

After the last commit, reply briefly:

- The commits created (short hash and subject), in order.
- The files left uncommitted and why (out of scope, possible secret, ambiguous).
