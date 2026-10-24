# CLAUDE.md

Guidance for agents adding or changing skills in this repository.

## Language

All repository content (code, docs, skills, manifests, commit messages) is written in English.

## Shared conventions

- Workflow artifacts (`spec.md`, `plan.md`, `todo.md`, `test-script.md`) live in `.sdd/<task-slug>/`, where `<task-slug>` is a short kebab-case task name. Skills that read or write them must use this location unless the user says otherwise.
- Every skill is documented in a section of `README.md` in the same change that adds it.

## Workflow and handoff

The skills are meant to be used in sequence to develop a task from scratch: `/issue` -> `/spec` -> `/plan` -> `/build` -> `/commit` -> `/review` -> `/pr` -> `/generate-tests` -> `/execute-tests`. Each skill suggests the next step at the end of its run.

To save context and tokens, every skill reads and writes `HANDOFF.md`:

- The file lives at the root of the git repository where the task started and is never committed. Skills must check that it is ignored (`git check-ignore -q HANDOFF.md`) and, if it is not, add it to the local exclude file given by `git rev-parse --git-path info/exclude`. `/commit` must never stage it, not even with `--all`.
- A skill reads it first, if it exists, and treats it as context to verify against the files, not as truth.
- A skill updates it last, with one section per task (`## <task-slug>`) containing: stage (skill run and date), what was done, decisions, artifacts (paths, issue and PR numbers, branch) and next steps with open questions. Outdated content is replaced, not appended, and each task section stays short (about 60 lines at most). It never contains secrets.
- Every new skill must include the same `## Handoff` section, tailored to what it does, and keep its `allowed-tools` (when set) able to write the file and run `git check-ignore` and `git rev-parse`.

## Skill frontmatter: `disable-model-invocation`

- Set `disable-model-invocation: false` in every new skill. This is the default: agents may invoke the skill on their own when the task calls for it.
- Use `disable-model-invocation: true` only as an exception, for skills that must run exclusively when the user explicitly invokes them (for example, skills with destructive or hard-to-reverse side effects).
- Always write the field explicitly, even when it is `false`, so the choice is visible.
- When a skill is set to `true`, state the reason in the skill's section of `README.md`.
