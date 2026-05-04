---
aliases:
  - Learning
tags:
  - dev/ai/llm
date: 2026-03-15
---
**Related:** [[Claude Code]], [[Distribute]], [[Commands]], [[Obsidian Syncing]], [[Hooks]]

---

## Description

Custom ``command`` that reads the last transcription created by the `hook` `Obsidian Syncing` and its related notes, and distributes the relevant content into one or more existing or new notes.

---

## Instructions

Read the latest unprocessed Claude Code session transcript from my Obsidian vault
and distribute its content into the relevant notes using MCPVault MCP tools.

## Steps

1. **Find the transcript.**
   - Use `list_directory` on `Journal/Claude Sessions/` → find the most recent date folder.
   - Use `list_directory` on that folder → list .md files.
   - Use `get_frontmatter` on the most recent file to confirm `distributed: false`.
   - If $ARGUMENTS is provided, filter by filename match instead.
   - Use `read_note` to get the full transcript content.

2. **Discover target notes.**
   - Parse the `Related:` wikilinks from the transcript header — these are the primary targets.
   - Extract 3–5 key technical topics from the conversation body.
   - Run `search_notes` with those topics to find additional relevant notes not in Related:.
   - Combine both sources, de-duplicate, cap at 10 total (prioritize Related: first).

3. **Read target notes in batch.**
   - Use `read_multiple_notes` with all target paths (up to 10 at once).
   - If more than 10, make a second call for the remainder.

4. **Write the Summary section in the transcript.**
   - Synthesize a 2–4 sentence summary of what was discussed and decided.
   - Use `patch_note` to replace `Write here...` inside `## Summary` with the summary text.

5. **Distribute content into each target note.**
   For each note, add only what's new and relevant:
   - **Existing sections** (Description, Details, Key concepts, etc.) →
     use `patch_note` to append new information at the end of the right section.
   - **New sections needed** → use `patch_note` to insert them before `## Claude Sessions`.
   - **Stub notes** (`#claude-generated`, body is `Write here...`) →
     use `patch_note` to replace the stub with a proper overview.
   - **Code examples** → add in fenced code blocks with language tags.

6. **Mark as distributed.**
   - Use `update_frontmatter` on the transcript to set `distributed: true`.

7. **Show a summary** of what was changed in each note.

## Rules

- NEVER remove or overwrite existing content — only add.
- Match each note's style: header levels, list formats, wikilink conventions, language.
- Don't duplicate information already present in the note.
- Keep additions concise — reference notes, not essays.
- Preserve all frontmatter, tags, and `## Claude Sessions` sections untouched.
- When using `patch_note`, use a string long enough to be unique in the file.
- If `read_multiple_notes` needs more than 10 files, make two calls.

---

## Claude Sessions
