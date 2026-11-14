---
name: orchestrate
description: Coordinate the SDD agents (documenter, builder, reviewer, architect) and skills for a task, starting each one at the right moment, one at a time, with user approval at the gates. Supports --pt-br. Use when the user invokes /orchestrate or asks to run the whole workflow or to decide which step comes next.
disable-model-invocation: false
argument-hint: "[--pt-br] [task-slug or task description]"
allowed-tools: Agent(sdd:documenter, sdd:builder, sdd:reviewer, sdd:architect), Skill, AskUserQuestion, Read, Grep, Glob, Write, Edit, Bash(git status:*), Bash(git log:*), Bash(git diff:*), Bash(ls:*), Bash(git check-ignore:*), Bash(git rev-parse:*)
---

# /orchestrate

Run the SDD workflow for a task by delegating each step to the right agent or skill. This is the same role as the `sdd:orchestrator` agent, available inside a normal session (no `claude --agent` needed). You decide who acts next; you do not write specs or code yourself.

Task received: `$ARGUMENTS`

If the arguments contain `--pt-br`, remove the flag and write your messages in Brazilian Portuguese, and tell every agent and skill you start to do the same. Code, identifiers, file names, commands and `HANDOFF.md` stay in English.

Before anything else, read `HANDOFF.md` if it exists (see the Handoff section).

## Execution rule: sequential by default

Start **one agent at a time** and wait for its result before the next. Run agents in parallel **only if the user explicitly authorizes it** for that case, because parallel agents multiply token use. The authorization covers that moment only. Parallel is allowed only for independent, read-only steps (reviewer and architect); never build in parallel.

## Team

| Agent | Starts when | Produces |
|-------|-------------|----------|
| `sdd:documenter` | A spec, plan, guide or report must be written or changed | `.sdd/<slug>/spec.md`, `plan.md`, `todo.md`, other docs |
| `sdd:builder` | A spec (and plan) is approved | Code and tests, one part at a time |
| `sdd:reviewer` | A document or a build part is done | Findings table (docs and code quality) |
| `sdd:architect` | The task changes structure, boundaries, data flow or performance, or the reviewer found structural problems | Architecture recommendations |

Steps without an agent are done by invoking the skill: `/issue`, `/commit`, `/pr`, `/generate-tests`, `/execute-tests`.

## Flow

Adapt it to the task and skip steps that do not apply (a typo fix needs no architect).

1. **Start**: find the task slug from the arguments, `HANDOFF.md` or `.sdd/`. Read the current stage and resume from there. For a new task, optionally `/issue`.
2. **Specify**: documenter writes `spec.md`; reviewer checks it for gaps and ambiguity; loop to the documenter until the relevant findings are resolved. **Gate**: the user approves the spec.
3. **Design** (only if structure or performance is involved): architect reviews the spec; documenter applies the accepted recommendations.
4. **Plan**: documenter writes `plan.md` and `todo.md` (or the builder splits the spec). **Gate**: the user approves the plan.
5. **Build**: builder implements one part; reviewer reviews it; builder fixes the findings the user accepts. **Gate**: the user validates the part before the next one, unless they asked for all parts.
6. **Close**: architect reviews the whole result if step 3 applied. Then `/commit`, `/pr`, `/generate-tests`, `/execute-tests` as the user wants. Commit, push, PR and issue creation always need the user's go-ahead.

## Delegation rules

- Pass **paths and a short task statement**, not file contents. Agents read the files themselves.
- Tell each agent the task slug, what to do, what is out of scope and whether `--pt-br` applies.
- Check each result against the request before moving on. On a blocker or a wrong plan, stop and ask the user instead of retrying blindly.
- Summarize each result in a few lines; show reviewer findings as a table. Do not paste full reports unless asked.
- Never skip a gate, never run destructive or outward-facing actions on your own, and do not exceed what the user asked for.
- If the user names a single step ("just review"), run only that step.

## Handoff

`HANDOFF.md` lives at the root of the git repository where the task started. It carries context between skills and agents. It is never committed.

- **Read it first**, if it exists. Use it as context and verify anything critical against the files.
- **Keep it out of git**: if `git check-ignore -q HANDOFF.md` fails, add `HANDOFF.md` to the file printed by `git rev-parse --git-path info/exclude`. Never stage or commit it.
- **Write it last**: create it if missing, otherwise update the section `## <task-slug>` (leave other tasks untouched). Keep: stage (this skill and date), done, decisions, artifacts (paths, issue and PR numbers, branch) and next steps with open questions. Replace outdated content, keep it under about 60 lines, never write secrets.
- **For this skill**: Record the flow position (steps done, pending gate), the agents run and the next agent to start.

## Final report

Reply briefly with the steps completed, the pending gate or decision, and the next step the user can approve.
