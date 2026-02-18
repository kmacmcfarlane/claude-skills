# claude-skills

A collection of [Claude Code skills](https://docs.anthropic.com/en/docs/claude-code/skills) (slash commands) for use across projects.

## Skills

| Skill | Description |
|---|---|
| [create-skill](skills/create-skill/) | Bootstrap a new skill from a description |

## Usage

### Per-project (recommended)

Copy or symlink individual skills into your project's `.claude/skills/` directory:

```bash
# Copy a skill
cp -r /path/to/claude-skills/skills/create-skill .claude/skills/

# Or symlink it
ln -s /path/to/claude-skills/skills/create-skill .claude/skills/create-skill
```

### Global (all projects)

Symlink skills into your personal Claude Code directory:

```bash
ln -s /path/to/claude-skills/skills/create-skill ~/.claude/skills/create-skill
```

## Creating new skills

Use the `create-skill` skill itself:

```
/create-skill A skill that runs the test suite and summarizes failures
```

## License

This project is licensed under the [GPL-3.0](LICENSE).
