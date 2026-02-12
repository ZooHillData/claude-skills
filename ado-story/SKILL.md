---
name: ado-story
description: Fetch an Azure DevOps user story and its child tasks, then generate a formatted markdown file.
user-invocable: true
argument-hint: <work-item-id> [output-path]
---

Fetch an Azure DevOps user story and all of its child tasks, then create a single markdown summary file.

## Arguments

- `$0` — the work item ID (required)
- `$1` — output file path (optional, defaults to `~/Downloads/user-story-$0.md`)

## Prerequisites

Source `~/.bashrc` before making API calls to load `ADO_API_KEY` and `ADO_API_HOST`.

## Steps

1. **Fetch the user story** with relations to discover child work items:

```bash
source ~/.bashrc; curl -s -u ":${ADO_API_KEY}" "${ADO_API_HOST}/_apis/wit/workitems/{id}" \
  -G -d "api-version=7.1" -d "$expand=relations"
```

2. **Extract child IDs** from `relations` where `rel` is `System.LinkTypes.Hierarchy-Forward` (name: "Child"). Parse the work item ID from each relation's `url`.

3. **Batch-fetch all child work items**:

```bash
source ~/.bashrc; curl -s -u ":${ADO_API_KEY}" "${ADO_API_HOST}/_apis/wit/workitems" \
  -G -d "api-version=7.1" -d "ids={id1},{id2},{id3}"
```

4. **Generate the markdown file** at the output path with the structure below.

## Markdown Output Format

```markdown
# User Story {id}: {title}

| Field | Value |
|-------|-------|
| **Type** | {work item type} |
| **State** | {state} |
| **Priority** | {priority} |
| **Assigned To** | {assigned to display name} |
| **Iteration** | {iteration path} |
| **Area Path** | {area path} |
| **Parent** | {parent id if present} |
| **Story Points** | {story points if present} |
| **QA Estimation** | {qa estimation if present} |
| **Created** | {date} by {creator} |
| **Changed** | {date} by {changer} |
| **ADO Link** | {html link from _links.html.href} |

(Only include rows where values exist)

## Description

​```markdown
{description text, converted from HTML to plain text/markdown}
​```

## Acceptance Criteria

​```markdown
{acceptance criteria, converted from HTML to plain text/markdown}
​```

## Attachments

(List any attached files by name, if present)

---

## Tasks

### Task {id}: {title}

| Field | Value |
|-------|-------|
| **State** | ... |
| **Priority** | ... |
| **Story Points** | ... |
| **Assigned To** | ... |
| **Developer** | ... |
| **Iteration** | ... |
| **QA Estimation** | ... |

#### Description

​```markdown
{description}
​```

#### Acceptance Criteria

​```markdown
{acceptance criteria, or "_No acceptance criteria defined._" if missing}
​```

---

(Repeat for each child task, ordered by title)
```

## Key Rules

- Convert HTML fields (description, acceptance criteria) to readable plain text/markdown
- Wrap description and acceptance criteria content in markdown codeblocks (triple backticks with `markdown` language tag) so they don't conflict with the document's own heading structure
- Sort tasks by their title (they often have numeric prefixes like "01", "02")
- Only include metadata rows in tables where values actually exist
- If a field like acceptance criteria is empty/missing, note it as "_No acceptance criteria defined._"
