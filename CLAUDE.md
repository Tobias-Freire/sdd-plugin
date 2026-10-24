# CLAUDE.md

Guidance for agents adding or changing skills in this repository.

## Language

All repository content (code, docs, skills, manifests, commit messages) is written in English.

## Skill frontmatter: `disable-model-invocation`

- Set `disable-model-invocation: false` in every new skill. This is the default: agents may invoke the skill on their own when the task calls for it.
- Use `disable-model-invocation: true` only as an exception, for skills that must run exclusively when the user explicitly invokes them (for example, skills with destructive or hard-to-reverse side effects).
- Always write the field explicitly, even when it is `false`, so the choice is visible.
- When a skill is set to `true`, state the reason in the skill's section of `README.md`.
