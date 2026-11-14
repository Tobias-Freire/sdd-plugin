# sdd-plugin

Personal Claude Code plugin with skills and context that guide the agent through recurring tasks.

## Structure

```
sdd-plugin/
├── .claude-plugin/
│   ├── plugin.json         # plugin manifest
│   └── marketplace.json    # lets the repository be installed as a marketplace
├── agents/
│   ├── architect.md        # Architect agent
│   ├── builder.md          # Builder agent
│   ├── documenter.md       # Documenter agent
│   ├── orchestrator.md     # Orchestrator agent
│   └── reviewer.md         # Reviewer agent
├── skills/
│   ├── build/SKILL.md           # /build skill
│   ├── commit/SKILL.md          # /commit skill
│   ├── docs/SKILL.md            # /docs skill
│   ├── execute-tests/SKILL.md   # /execute-tests skill
│   ├── generate-tests/SKILL.md  # /generate-tests skill
│   ├── issue/SKILL.md           # /issue skill
│   ├── orchestrate/SKILL.md     # /orchestrate skill
│   ├── plan/SKILL.md            # /plan skill
│   ├── pr/SKILL.md              # /pr skill
│   ├── review/SKILL.md          # /review skill
│   └── spec/SKILL.md            # /spec skill
├── CLAUDE.md
└── README.md
```

When it makes sense, also add `references/` (supporting material shared between skills).

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

Each skill is described in detail in its own `SKILL.md` (arguments, behavior and examples). Add new skills to this table.

| Skill | Details |
|-------|---------|
| `/issue` | [skills/issue/SKILL.md](skills/issue/SKILL.md) |
| `/spec` | [skills/spec/SKILL.md](skills/spec/SKILL.md) |
| `/plan` | [skills/plan/SKILL.md](skills/plan/SKILL.md) |
| `/build` | [skills/build/SKILL.md](skills/build/SKILL.md) |
| `/commit` | [skills/commit/SKILL.md](skills/commit/SKILL.md) |
| `/review` | [skills/review/SKILL.md](skills/review/SKILL.md) |
| `/pr` | [skills/pr/SKILL.md](skills/pr/SKILL.md) |
| `/generate-tests` | [skills/generate-tests/SKILL.md](skills/generate-tests/SKILL.md) |
| `/execute-tests` | [skills/execute-tests/SKILL.md](skills/execute-tests/SKILL.md) |
| `/docs` | [skills/docs/SKILL.md](skills/docs/SKILL.md) |
| `/orchestrate` | [skills/orchestrate/SKILL.md](skills/orchestrate/SKILL.md) |

Artifacts of the workflow (`spec.md`, `plan.md`, `todo.md`, `test-script.md`) are saved in `.sdd/<task-slug>/`.

> Plugin skills are namespaced under the plugin name, so the guaranteed form is `/sdd:commit`. Type `/` in Claude Code to see how it appears in your installation.

## Agents

Subagents in `agents/`, each with a narrow role. They reuse the skills above (preloaded through the `skills` field) and read and write `HANDOFF.md` like the skills do. They work **one at a time**; parallel runs only happen when the user explicitly authorizes them, because they multiply token use.

| Agent | Role | Writes |
|-------|------|--------|
| `sdd:documenter` | Writes and edits specs, plans, guides, analyses and reports. Fixed set of document types; destinations: `.txt`, Markdown, Google Docs or Claude Artifact. | Documents only |
| `sdd:builder` | Takes the spec, splits it into parts (or follows `plan.md`) and implements one part at a time with unit tests. | Code and tests |
| `sdd:reviewer` | Reviews documents for gaps and semantic problems, and code for testability, readability, maintainability and modularity. Reports findings in a table. | `HANDOFF.md` only |
| `sdd:architect` | Reviews documents and code for consistency, scalability, performance and maintainability and recommends changes. Runs on Opus. | `HANDOFF.md` (and optionally `architecture.md`) |
| `sdd:orchestrator` | Activates the other agents and the skills at the right moments, with user gates (spec, plan, each build part). | `HANDOFF.md` only |

Subagents cannot start other subagents, so the Orchestrator must run as the **main-session agent**:

```bash
claude --agent sdd:orchestrator
```

To make it the default for a project, set `"agent": "sdd:orchestrator"` in `.claude/settings.json`. The other four can be requested by name in any session (for example, "use the sdd:reviewer agent on this diff").

`/orchestrate` offers the same coordination without restarting the session.

The Documenter's Google Docs, Google Drive and Claude Docs access requires those connectors to be authorized.
