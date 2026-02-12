---
name: save-plan
description: Save the current plan or conversation context to a dated plan file. This skill should be used when the user asks to "save this plan", "archive the plan", "write the plan to a file", or uses /save-plan.
user-invocable: true
argument-hint: <slug> [content-source]
---

Save the current plan to a numbered markdown file inside a dated `plans/` directory.

## Arguments

- `$0` — a short kebab-case slug describing the plan (required). Examples: `global-editor`, `auth-refactor`, `query-migration`
- `$1` — where to source the plan content (optional). If omitted, use the plan content from the current conversation. Can be a file path to copy from, or `"conversation"` to extract from context.

## Output Path Convention

Plans are saved to: `./plans/YYYYMMDD/NN-slug.md`

- `YYYYMMDD` — today's date (e.g., `20260211`)
- `NN` — zero-padded sequential number, incrementing from existing files in today's folder
- `slug` — the `$0` argument

## Steps

1. **Determine today's date** in `YYYYMMDD` format.

2. **Ensure the directory exists**: `./plans/YYYYMMDD/`

3. **Determine the next sequence number**:
   - List existing files in `./plans/YYYYMMDD/`
   - Find the highest numeric prefix (e.g., if `01-foo.md` and `02-bar.md` exist, next is `03`)
   - If the directory is empty or doesn't exist, start at `01`
   - Zero-pad to 2 digits

4. **Identify the plan content**:
   - If `$1` is a file path, read that file as the content
   - Otherwise, extract the plan from the current conversation. Look for the most recent plan content — this is typically a markdown document with headings like "## Context", "## Approach", "## Files to Create", "## Verification", etc. It may have been provided by the user, generated during plan mode, or discussed in the conversation.

5. **Write the file** to `./plans/YYYYMMDD/NN-slug.md` with the plan content.

6. **Confirm** by telling the user the file path that was created.

## Key Rules

- Always use the project's working directory for the `./plans/` folder (not home directory)
- The slug should be kebab-case, lowercase, no spaces
- If the user provides a slug with spaces or other formatting, convert it to kebab-case
- Do not include the `plans/` prefix or date in the slug — those are added automatically
- If no clear plan content exists in the conversation, ask the user what content to save
