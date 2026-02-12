# Claude Skills

Custom [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills for the ZooHill Data team.

## What are skills?

Skills are reusable prompts that extend Claude Code with custom slash commands. Each skill lives in its own directory with a `SKILL.md` file that defines its name, description, and instructions.

## Installation

Clone this repo and symlink (or copy) individual skill directories into your local Claude Code skills folder:

```bash
git clone https://github.com/ZooHillData/claude-skills.git
```

To install a specific skill:

```bash
# Symlink approach (stays in sync with git pulls)
ln -s /path/to/claude-skills/save-plan ~/.claude/skills/save-plan

# Or copy approach
cp -r /path/to/claude-skills/save-plan ~/.claude/skills/save-plan
```

Skills are picked up automatically — restart Claude Code or start a new conversation to use them.

## Usage

Once installed, invoke a skill with its slash command:

```
/save-plan global-editor
/publish-plan save-plan
/ado-story 12345
```

## Publishing skills

Skills are published to this repo individually using the `publish-plan` skill:

```
/publish-plan <skill-name>
```

This copies the named skill from `~/.claude/skills/<skill-name>/` into this repo, commits, and pushes.

## Skill structure

Each skill is a directory containing at minimum a `SKILL.md` file:

```
skill-name/
├── SKILL.md          # Required — frontmatter + instructions
├── references/       # Optional — detailed documentation
├── examples/         # Optional — working examples
└── scripts/          # Optional — helper scripts
```

The `SKILL.md` frontmatter:

```yaml
---
name: skill-name
description: When to invoke this skill.
user-invocable: true
argument-hint: <required-arg> [optional-arg]
---
```
