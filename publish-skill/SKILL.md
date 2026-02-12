---
name: publish-skill
description: Publish a specific Claude Code skill to this repo. This skill should be used when the user asks to "publish a skill", "push skill to GitHub", "sync a skill", or uses /publish-skill.
user-invocable: true
argument-hint: <skill-name> [commit-message]
---

Copy a specific skill from the local `~/.claude/skills/` directory to the `zoohilldata/claude-skills` GitHub repository, then commit and push.

## Arguments

- `$0` — name of the skill directory to publish (required). Must match a directory name under `~/.claude/skills/`. Examples: `save-plan`, `ado-story`, `publish-skill`
- `$1` — commit message (optional). If omitted, auto-generate a message like `update skill: save-plan` or `add skill: new-skill-name`.

## Repository

- **Remote**: `https://github.com/zoohilldata/claude-skills`

## Steps

1. **Validate the skill exists**: Check that `~/.claude/skills/$0/SKILL.md` exists. If not, list the available skills under `~/.claude/skills/` and ask the user which one to publish.

2. **Locate the local repo clone**. The repo path is cached in `~/.claude/skills/publish-skill/.repo-path` to avoid re-searching every time.
   - **First**, check if `~/.claude/skills/publish-skill/.repo-path` exists and contains a valid path (the directory has a `.git` folder). If so, use that path as `<repo-path>`.
   - **If not cached or invalid**, search for an existing clone: `find ~ -maxdepth 4 -type d -name "claude-skills" 2>/dev/null` and check each result for a `.git` directory with the correct remote (`git -C <path> remote get-url origin` should contain `zoohilldata/claude-skills`).
   - If found, save the path to `~/.claude/skills/publish-skill/.repo-path` and use it as `<repo-path>`.
   - If **not found**, ask the user where they'd like to clone it using the AskUserQuestion tool. Suggest a sensible default. Then clone with `gh repo clone zoohilldata/claude-skills <chosen-path>` and save the path to `~/.claude/skills/publish-skill/.repo-path`.

3. **Pull latest** from the remote to avoid conflicts:
   ```bash
   git -C <repo-path> pull --rebase
   ```

4. **Copy the specific skill**: Sync only the named skill directory into the repo, replacing existing content for that skill:
   ```bash
   rsync -av --delete ~/.claude/skills/$0/ <repo-path>/$0/
   ```
   This overwrites the skill in the repo with the local version. Other skills in the repo are left untouched.

5. **Update the README skills table**: Read `<repo-path>/README.md` and find the `## Skills` markdown table. Extract the skill's description from the `description` field in the `SKILL.md` frontmatter, and the command from the `name` and `argument-hint` fields.
   - If the skill already has a row in the table, update it with the current description and argument hint.
   - If the skill is not in the table, add a new row in alphabetical order by skill name.
   - Table format: `| **skill-name** | \`/skill-name <argument-hint>\` | description |`

6. **Check for changes**: Run `git status` in the repo. If there are no changes, inform the user that the skill is already up to date and stop.

7. **Stage all changes** (skill files and README):
   ```bash
   git -C <repo-path> add $0/ README.md
   ```

8. **Commit**:
   - If `$1` was provided, use it as the commit message
   - Otherwise, check if the skill directory is new to the repo (`git diff --cached --diff-filter=A`) and format accordingly:
     - New skill: `add skill: $0`
     - Updated skill: `update skill: $0`

9. **Push** to the remote:
   ```bash
   git -C <repo-path> push
   ```

10. **Confirm** by telling the user the commit hash and the skill that was published.

## Key Rules

- Only copy the specific named skill, not all skills
- Always pull before syncing to avoid push conflicts
- Use `rsync --delete` on the individual skill directory so removed files within the skill are cleaned up
- Do not push if there are no changes
- The repo should contain skill directories at its root (e.g., `save-plan/SKILL.md`, `ado-story/SKILL.md`)
