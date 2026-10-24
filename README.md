# sdd-plugin

Personal Claude Code plugin with skills and context that guide the agent through recurring tasks.

## Structure

```
sdd-plugin/
├── .claude-plugin/
│   ├── plugin.json         # plugin manifest
│   └── marketplace.json    # lets the repository be installed as a marketplace
├── skills/
│   └── commit/
│       └── SKILL.md        # /commit skill
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

## Skills

### `/commit`

Creates atomic commits following Conventional Commits from the changes made in the current session.

- Files from different parts of the task go into separate commits.
- Modified files that do not belong to the current session are not committed, unless the request explicitly asks to commit everything (`--all` or equivalent text).
- By default: English message, subject line only, no body and no AI co-authorship.

| Argument | Effect |
|----------|--------|
| `--with-body` | Adds a body to the commit message. |
| `--with-ai` | Adds a body and AI co-authorship (`Co-Authored-By`). |
| `--pt-br` | Writes the message in Brazilian Portuguese. |
| `--all` | Also includes pending changes that are not from the session. |

Arguments can be combined:

```
/commit
/commit --with-body
/commit --with-ai --pt-br
/commit --with-body --pt-br
```

### `/spec`

Generates a detailed specification (`spec.md`) for a task: goals, non-goals, testable requirements, flows, edge cases, technical approach and acceptance criteria.

```
/spec add rate limiting to the public API
```

### `/plan`

Breaks the specification into independent, executable parts. Generates `plan.md` (each part with goal, requirements covered, approach, tests and verification) and `todo.md` (the checklist of steps and checks for each part).

```
/plan
/plan rate-limiting
```

### `/build`

Builds the task from `spec.md`, `plan.md` and `todo.md`, one part at a time, and waits for the user's validation after each part. When the task involves code, the agent must add at least unit tests for every method and follow clean code principles.

| Argument | Effect |
|----------|--------|
| *(none)* | Builds the next unfinished part and waits for validation. |
| `--validate` | Runs the checks and tests of the current part. |
| `--all` | Builds all parts and runs all checks and tests without waiting for validation between parts. |

```
/build
/build --validate
/build --all
```

### `/generate-tests`

Generates a manual test script (`test-script.md`) that covers all scenarios of a feature or task: happy paths, alternative flows, validation, boundary values, errors, security and regression. Each test case has preconditions, concrete steps and an expected result, and is mapped to the spec requirements.

```
/generate-tests
/generate-tests rate-limiting
```

### `/execute-tests`

Executes the manual test script (`test-script.md`) on its own, using tools such as Playwright for UI flows and HTTP requests for APIs. Records pass, fail or blocked for each test case with evidence and reports the results. It only runs against local, development or staging environments.

```
/execute-tests
/execute-tests rate-limiting
```

Artifacts of the workflow (`spec.md`, `plan.md`, `todo.md`, `test-script.md`) are saved in `.sdd/<task-slug>/`.

> Plugin skills are namespaced under the plugin name, so the guaranteed form is `/sdd:commit`. Type `/` in Claude Code to see how it appears in your installation.
