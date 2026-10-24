---
name: docs
description: Create or edit a document of any kind (process documentation, code analysis, research, article, report, guide) as a Markdown file by default, or in an external tool such as Google Docs when a link or destination is given. Supports --pt-br. Use when the user invokes /docs or asks to write, document, update or edit a document.
disable-model-invocation: false
argument-hint: "[--pt-br] <what to document> [link or destination]"
allowed-tools: Read, Grep, Glob, Write, Edit, WebFetch, Bash(git log:*), Bash(git status:*), Bash(ls:*), Bash(git check-ignore:*), Bash(git rev-parse:*)
---

# /docs

Create a new document or edit an existing one, in the format and place the user asks for. The kind of document (process documentation, code analysis, research, article, report, tutorial, ...) is decided from the task, not from a fixed template.

Task received: `$ARGUMENTS`

If the arguments contain `--pt-br`, remove the flag from them and write the document in Brazilian Portuguese instead of English (headings included). Code, identifiers, file names, commands and `HANDOFF.md` stay in their original form (English).

Before anything else, read `HANDOFF.md` if it exists (see the Handoff section).

## 1. Understand the task

1. If the arguments are empty, use the task discussed in the conversation. If there is none, ask the user what to document.
2. Identify, from the request:
   - **Document type**: process, code analysis, research, article, report, guide, etc. If it is unclear, pick the closest type and record the choice in the final report.
   - **Audience and purpose**: who will read it and what they must be able to do or decide afterwards. Default to a reader with no prior context.
   - **Destination**: see section 2.
   - **Mode**: create a new document or edit an existing one.
3. Gather the material. Read the code, files, existing docs or sources the document depends on (use `WebFetch` for public links). Never invent facts: if something cannot be verified, mark it as an open question in the document or in the final report.
4. Ask the user only about decisions you cannot resolve from the request or the sources. Prefer sensible defaults.

## 2. Choose the destination

Decide in this order:

1. **The user names a place** (a path, a folder, another tool): use it.
2. **The arguments contain a link** to a documentation tool (Google Docs, Notion, Confluence, Claude Docs, etc.): the document goes there. See "External destinations".
3. **Otherwise**: create a Markdown file at `docs/<doc-slug>.md` in the repository root, where `<doc-slug>` is a short kebab-case name. Create the directory if needed. If the repository already keeps documentation elsewhere (for example `documentation/` or `doc/`), follow that convention.

If the target file already exists, read it and edit it instead of overwriting it. Preserve what is still correct, its structure and its style; change only what the task requires.

## 3. External destinations

Use the tools available in the session (connectors such as Google Drive or Claude Docs, or the browser tools). If a Google Workspace skill is available, invoke it before the first call that creates or changes a Google file. If the needed connector is not authorized or no tool can reach the destination, stop and tell the user what to authorize; do not silently fall back to a local file. Offer the Markdown file as an alternative and wait for their answer.

**Existing document** (the link points to a document that already has content):

- Read it first, including its formatting: heading levels, fonts, sizes, colors, alignment, list and table styles, spacing, header and footer.
- Reproduce that formatting exactly in what you add or change. Do not restyle, reflow or reorder parts the task does not touch.
- Prefer targeted edits (insert, replace a range) over rewriting the whole document.

**New document** (empty or newly created):

- Use a clear heading hierarchy (title, `Heading 1`, `Heading 2`) with a consistent font, size and spacing.
- Center the elements that stand alone: title, images, diagrams, captions and tables. Text paragraphs and lists stay left-aligned for readability.
- Keep widths consistent: tables span the same width as the text area, images and diagrams share a similar width, and no element is much narrower or wider than the others. Avoid tables with one very wide column next to very narrow ones.
- Use the tool's native structures (headings, lists, tables) instead of simulating them with spaces or manual line breaks.
- When the tool allows it, check the result after writing (read it back or take a screenshot) and fix visible problems such as misalignment or overflowing tables.

## 4. Write the document

Choose the structure that fits the type. Typical shapes, adapted as needed:

- **Process**: purpose, scope, prerequisites, numbered steps, roles, expected results, troubleshooting.
- **Code analysis**: scope, architecture overview, key components, flows, findings (with file references), risks, recommendations.
- **Research**: question, method and sources, findings, comparison, conclusion, references.
- **Article**: title, introduction with the main point, body in logical sections, conclusion.
- **Report**: summary, context, data or results, analysis, conclusions, next steps.
- **Guide or tutorial**: goal, prerequisites, steps with examples, common mistakes.

Start with a short summary or introduction that tells the reader what the document covers and why it matters.

Writing rules, valid for every document, even technical ones:

- **Clear and easy to understand for any audience**: plain words, short sentences, one idea per paragraph. Define each technical term or acronym the first time it appears. Explain the "why", not only the "what".
- **Detailed**: include the concrete information the reader needs (steps, values, examples, file paths, commands, sources). A reader should not have to guess or ask.
- **Objective and concise**: no filler, repetition, vague claims or marketing tone. If a sentence does not help the reader, remove it.
- Use lists for sequences and options, tables for comparisons, code blocks for commands and code, and diagrams or examples when they explain better than text.
- Keep a consistent tone, terminology and heading style throughout.
- Cite the source of data, quotes and claims that come from outside the repository.

## Handoff

`HANDOFF.md` lives at the root of the git repository where the task started. It carries context between skills, so the next step does not have to rediscover it. It is never committed.

- **Read it first**, if it exists. Use it as context and verify anything critical against the files.
- **Keep it out of git**: if `git check-ignore -q HANDOFF.md` fails, add `HANDOFF.md` to the file printed by `git rev-parse --git-path info/exclude`. Never stage or commit it, not even when everything pending is committed.
- **Write it last**: create it if missing, otherwise update the section `## <task-slug>` (one section per task; leave other tasks untouched). Keep these parts: stage (this skill and date), done, decisions, artifacts (paths, issue and PR numbers, branch) and next steps with open questions. Replace outdated content instead of appending, keep the section under about 60 lines and never write secrets.
- **For this skill**: Record the document type, its path or link, whether it was created or edited, the main sources used and the open questions. Next step: review the document with the user, then `/commit` if it is a file in the repository.

## 5. Final report

Reply briefly with the document type, where it was saved (path or link), whether it was created or edited, the assumptions made and any open questions. If the document is a file in the repository, suggest `/commit` as the next step. Do not paste the whole document in the reply.
