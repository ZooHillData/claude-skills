---
name: publish-plan
description: Publish all local Claude Code skills to the zoohilldata/claude-skills GitHub repository. This skill should be used when the user asks to "publish skills", "push skills to GitHub", "sync skills", or uses /publish-plan.
user-invocable: true
argument-hint: [commit-message]
---

Sync all skills from the local `~/.claude/skills/` directory to the `zoohilldata/claude-skills` GitHub repository, then commit and push.

## Arguments

- `$0` — commit message (optional). If omitted, auto-generate a message describing which skills were added or updated.

## Repository

- **Remote**: `https://github.com/zoohilldata/claude-skills`
- **Local clone**: `~/Documents/code/nrg/research-platform-ui/claude-skills` (default location)

If the local clone does not exist, clone it first:
```bash
gh repo clone zoohilldata/claude-skills ~/Documents/code/nrg/research-platform-ui/claude-skills
```

## Steps

1. **Locate the local repo clone**. Check if `~/Documents/code/nrg/research-platform-ui/claude-skills/.git` exists. If not, clone the repo there.

2. **Pull latest** from the remote to avoid conflicts:
   ```bash
   git -C <repo-path> pull --rebase
   ```

3. **Sync skills**: Copy every skill directory from `~/.claude/skills/` into the repo root, replacing existing content:
   ```bash
   rsync -av --delete ~/.claude/skills/ <repo-path>/ --exclude '.git'
   ```
   This ensures the repo mirrors the local skills directory exactly — new skills are added, updated skills are overwritten, and removed skills are deleted.

4. **Check for changes**: Run `git status` in the repo. If there are no changes, inform the user that everything is already up to date and stop.

5. **Stage all changes**:
   ```bash
   git -C <repo-path> add -A
   ```

6. **Commit**:
   - If `$0` was provided, use it as the commit message
   - Otherwise, generate a message by inspecting `git diff --cached --name-only` to list which skills were changed. Format: `update skills: save-plan, publish-plan` or `add skill: new-skill-name`

7. **Push** to the remote:
   ```bash
   git -C <repo-path> push
   ```

8. **Confirm** by telling the user the commit hash and listing which skills were published.

## Key Rules

- Always pull before syncing to avoid push conflicts
- Use `rsync --delete` so the repo stays in sync (skills removed locally are removed from the repo)
- Exclude the `.git` directory from the rsync
- Do not push if there are no changes
- The repo should contain only skill directories at its root (e.g., `save-plan/SKILL.md`, `ado-story/SKILL.md`) — no extra wrapper directories
