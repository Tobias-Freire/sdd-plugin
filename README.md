# sdd-plugin

Personal Claude Code plugin with skills and context that guide the agent through recurring tasks.

## Structure

```
sdd-plugin/
├── .claude-plugin/
│   ├── plugin.json         # plugin manifest
│   └── marketplace.json    # lets the repository be installed as a marketplace
├── skills/
│   ├── build/SKILL.md           # /build skill
│   ├── commit/SKILL.md          # /commit skill
│   ├── execute-tests/SKILL.md   # /execute-tests skill
│   ├── generate-tests/SKILL.md  # /generate-tests skill
│   ├── issue/SKILL.md           # /issue skill
│   ├── plan/SKILL.md            # /plan skill
│   ├── pr/SKILL.md              # /pr skill
│   ├── review/SKILL.md          # /review skill
│   └── spec/SKILL.md            # /spec skill
├── CLAUDE.md
└── README.md
```

When it makes sense, also add `agents/` (subagents) and `references/` (supporting material shared between skills).

## Installation

Directly from GitHub:

```
/plugin marketplace add Tobias-Freire/sdd-plugin
/plugin install sdd@sdd-marketplace
```

Or from a local clone:

```
/plugin marketplace add /path/to/sdd-plugin
/plugin install sdd@sdd-marketplace
```

To update after new changes:

```
/plugin marketplace update sdd-marketplace
```

Or, directly from the terminal, without opening a Claude Code session:

```bash
claude plugin marketplace update sdd-marketplace && claude plugin update sdd@sdd-marketplace
```

Restart any open Claude Code session afterwards to load the updated skills.

> `plugin update` compares the `version` in `.claude-plugin/plugin.json`. If new skills do not show up after updating, check that the version was bumped; as a workaround, run `claude plugin uninstall sdd@sdd-marketplace && claude plugin install sdd@sdd-marketplace`.

## Workflow

The skills are meant to be used in sequence to develop a task from scratch:

```
/issue -> /spec -> /plan -> /build -> /commit -> /review -> /pr -> /generate-tests -> /execute-tests
```

Each skill suggests the next step when it finishes. To save context and tokens, the skills record what was done and the next steps in a `HANDOFF.md` file at the root of the repository where the task started, and read it when the next skill starts. This file is local: it is never committed (the skills add it to the repository's ignore rules if needed, and `/commit` never stages it).

## Skills

### `/commit`

Creates atomic commits following Conventional Commits from the changes made in the current session.

- Files from different parts of the task go into separate commits.
- Modified files that do not belong to the current session are not committed, unless the request explicitly asks to commit everything (`--all` or equivalent text).
- By default: English message, subject line only, no body and no AI co-authorship.
- By default, nothing is pushed. The commits are pushed only with `--with-push` or when the request explicitly asks for it (for example, "commit and push").

| Argument | Effect |
|----------|--------|
| `--with-body` | Adds a body to the commit message. |
| `--with-ai` | Adds a body and AI co-authorship (`Co-Authored-By`). |
| `--pt-br` | Writes the message in Brazilian Portuguese. |
| `--all` | Also includes pending changes that are not from the session. |
| `--with-push` | Pushes the commits to the remote after committing. |

Arguments can be combined:

```
/commit
/commit --with-body
/commit --with-ai --pt-br
/commit --with-body --pt-br
/commit --with-push
```

### `/spec`

Generates a detailed specification (`spec.md`) for a task: goals, non-goals, testable requirements, flows, edge cases, technical approach and acceptance criteria.

```
/spec add rate limiting to the public API
/spec --pt-br add rate limiting to the public API
```

Add `--pt-br` to write everything this skill generates in Brazilian Portuguese instead of English.

### `/plan`

Breaks the specification into independent, executable parts. Generates `plan.md` (each part with goal, requirements covered, approach, tests and verification) and `todo.md` (the checklist of steps and checks for each part).

```
/plan
/plan rate-limiting
/plan --pt-br
```

Add `--pt-br` to write everything this skill generates in Brazilian Portuguese instead of English.

### `/build`

Builds the task from `spec.md`, `plan.md` and `todo.md`, one part at a time, and waits for the user's validation after each part. When the task involves code, the agent must add at least unit tests for every method and follow clean code principles.

| Argument | Effect |
|----------|--------|
| *(none)* | Builds the next unfinished part and waits for validation. |
| `--validate` | Runs the checks and tests of the current part. |
| `--all` | Builds all parts and runs all checks and tests without waiting for validation between parts. |
| `--pt-br` | Writes messages and reports in Brazilian Portuguese (code stays in English). |

```
/build
/build --validate
/build --all
/build --pt-br
```

### `/generate-tests`

Generates a manual test script (`test-script.md`) that covers all scenarios of a feature or task: happy paths, alternative flows, validation, boundary values, errors, security and regression. Each test case has preconditions, concrete steps and an expected result, and is mapped to the spec requirements.

```
/generate-tests
/generate-tests rate-limiting
/generate-tests --pt-br
```

Add `--pt-br` to write everything this skill generates in Brazilian Portuguese instead of English.

### `/execute-tests`

Executes the manual test script (`test-script.md`) on its own, using tools such as Playwright for UI flows and HTTP requests for APIs. Records pass, fail or blocked for each test case, with evidence, in `test-report.md`. It only runs against local, development or staging environments.

| Argument | Effect |
|----------|--------|
| *(none)* | Runs the tests and writes `test-report.md`. |
| `--add-to-pr` | Also posts the results as a comment on the task's open PR: first a table listing and describing the tests, then a bullet list with the result of each one and its evidence (text, images or videos). |
| `--add-to-issue` | Same comment, posted on the task's issue instead of the PR. |
| `--pt-br` | Writes the report and the comment in Brazilian Portuguese. |

```
/execute-tests
/execute-tests --add-to-pr
/execute-tests --add-to-issue
/execute-tests rate-limiting
/execute-tests --add-to-pr --pt-br
```

### `/review`

Reviews everything that was done in a task, looking for improvements, security flaws, optimizations and test gaps. It first lists the findings in a table with the location, why it needs a fix, the suggested fix and a priority (Low, Medium, High or Urgent), then asks which ones to fix. It does not change code.

```
/review
/review rate-limiting
/review 42
/review --pt-br
```

Add `--pt-br` to write everything this skill generates in Brazilian Portuguese instead of English.

### `/issue`

Creates a GitHub issue in the task's repository with appropriate labels (chosen from the ones that already exist) and a detailed description. It checks for duplicates first and shows the content for confirmation before creating it. Requires the GitHub CLI (`gh`) authenticated.

```
/issue add rate limiting to the public API
/issue rate-limiting
/issue --pt-br add rate limiting to the public API
```

Add `--pt-br` to write everything this skill generates in Brazilian Portuguese instead of English.

### `/pr`

Generates the title and description of a pull request from the branch's commits, diff and task artifacts, asks the user for approval and only then opens the PR on GitHub. If there is a related issue in the context, the PR is linked to it (`Closes #<issue>`). Requires the GitHub CLI (`gh`) authenticated.

```
/pr
/pr main
/pr main 42
/pr --pt-br
```

Add `--pt-br` to write everything this skill generates in Brazilian Portuguese instead of English.

Artifacts of the workflow (`spec.md`, `plan.md`, `todo.md`, `test-script.md`) are saved in `.sdd/<task-slug>/`.

> Plugin skills are namespaced under the plugin name, so the guaranteed form is `/sdd:commit`. Type `/` in Claude Code to see how it appears in your installation.
