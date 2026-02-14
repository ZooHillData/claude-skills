---
name: save-output
description: Save a long report or generated content from the conversation to a markdown file in ~/.scratch/claude/. Use when the user asks to "save that output", "export the report", "write that to a file", or uses /save-output.
user-invocable: true
argument-hint: [description of which content]
---

Save the most recent long report, analysis, or generated content from the conversation to a markdown file in `~/.scratch/claude/`.

## Arguments

- `$ARGUMENTS` — an optional description of which content to save (e.g., `"the API comparison table"`, `"the refactor summary"`). If omitted, use the most recent long-form output in the conversation. If provided, search the conversation for the output that best matches the description.

## Steps

1. **Ensure the output directory exists**: `~/.scratch/claude/`

2. **Identify the content to save**:
   - If `$ARGUMENTS` is provided, scan the conversation for the output that best matches the description. Look for long-form content (reports, tables, analyses, code reviews, summaries, generated documents, etc.) that relates to the user's description.
   - If `$ARGUMENTS` is empty or not provided, find the **most recent** long-form output in the conversation. "Long-form" means substantial generated content — typically more than a few sentences. Look for reports, analyses, summaries, tables, code blocks, generated documents, etc. Skip short conversational replies.
   - If multiple candidates exist and the description is ambiguous, ask the user which one they mean using AskUserQuestion.
   - If no suitable content is found, tell the user there's nothing to save and ask what they'd like exported.

3. **Generate a descriptive filename**:
   - Format: `YYYYMMDD-HHmm-slug.md`
   - `YYYYMMDD-HHmm` — current date and time (e.g., `20260213-1430`)
   - `slug` — a short kebab-case description of the content (3-5 words max), derived from:
     - The `$ARGUMENTS` if provided (e.g., `"API comparison"` → `api-comparison`)
     - Otherwise, infer from the content's topic/title/headings
   - Examples: `20260213-1430-api-comparison.md`, `20260213-0915-tracking-refactor-plan.md`

4. **Write the file** to `~/.scratch/claude/YYYYMMDD-HHmm-slug.md` with the identified content. Preserve the original markdown formatting. Do not wrap the content in additional markup or add headers that weren't in the original.

5. **Confirm** by telling the user the full file path that was written.

## Key Rules

- Always use `~/.scratch/claude/` as the output directory — create it if it doesn't exist
- Filenames must be kebab-case, lowercase, no spaces
- Preserve the original formatting of the content exactly as it appeared
- If the content was generated as part of a tool result or code block, extract just the meaningful content
- Do not add preamble like "Here is the saved report:" — just save the content as-is
- The timestamp in the filename uses 24-hour time
