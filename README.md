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

> Plugin skills are namespaced under the plugin name, so the guaranteed form is `/sdd:commit`. Type `/` in Claude Code to see how it appears in your installation.
