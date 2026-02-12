---
name: publish-plan
description: Publish a specific Claude Code skill to the zoohilldata/claude-skills GitHub repository. This skill should be used when the user asks to "publish a skill", "push skill to GitHub", "sync a skill", or uses /publish-plan.
user-invocable: true
argument-hint: <skill-name> [commit-message]
---

Copy a specific skill from the local `~/.claude/skills/` directory to the `zoohilldata/claude-skills` GitHub repository, then commit and push.

## Arguments

- `$0` — name of the skill directory to publish (required). Must match a directory name under `~/.claude/skills/`. Examples: `save-plan`, `ado-story`, `publish-plan`
- `$1` — commit message (optional). If omitted, auto-generate a message like `update skill: save-plan` or `add skill: new-skill-name`.

## Repository

- **Remote**: `https://github.com/zoohilldata/claude-skills`
- **Local clone**: `~/Documents/code/claude-skills`

If the local clone does not exist, clone it first:
```bash
gh repo clone zoohilldata/claude-skills ~/Documents/code/claude-skills
```

## Steps

1. **Validate the skill exists**: Check that `~/.claude/skills/$0/SKILL.md` exists. If not, list the available skills under `~/.claude/skills/` and ask the user which one to publish.

2. **Locate the local repo clone**. Check if `~/Documents/code/claude-skills/.git` exists. If not, clone the repo there.

3. **Pull latest** from the remote to avoid conflicts:
   ```bash
   git -C <repo-path> pull --rebase
   ```

4. **Copy the specific skill**: Sync only the named skill directory into the repo, replacing existing content for that skill:
   ```bash
   rsync -av --delete ~/.claude/skills/$0/ <repo-path>/$0/
   ```
   This overwrites the skill in the repo with the local version. Other skills in the repo are left untouched.

5. **Check for changes**: Run `git status` in the repo. If there are no changes, inform the user that the skill is already up to date and stop.

6. **Stage the skill's changes**:
   ```bash
   git -C <repo-path> add $0/
   ```

7. **Commit**:
   - If `$1` was provided, use it as the commit message
   - Otherwise, check if the skill directory is new to the repo (`git diff --cached --diff-filter=A`) and format accordingly:
     - New skill: `add skill: $0`
     - Updated skill: `update skill: $0`

8. **Push** to the remote:
   ```bash
   git -C <repo-path> push
   ```

9. **Confirm** by telling the user the commit hash and the skill that was published.

## Key Rules

- Only copy the specific named skill, not all skills
- Always pull before syncing to avoid push conflicts
- Use `rsync --delete` on the individual skill directory so removed files within the skill are cleaned up
- Do not push if there are no changes
- The repo should contain skill directories at its root (e.g., `save-plan/SKILL.md`, `ado-story/SKILL.md`)
