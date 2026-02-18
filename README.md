# claude-skills

A collection of [Claude Code skills](https://docs.anthropic.com/en/docs/claude-code/skills) (slash commands) for use across projects.

## Skills

| Skill | Description |
|---|---|
| [create-skill](skills/create-skill/) | Bootstrap a new skill from a description |
| [goa](skills/goa/) | Design-first API development with the Goa v3 framework for Go |

## Installation

### Global — all projects (recommended)

Installs each skill as an individual symlink in `~/.claude/skills/`, making them available in every project. Safe to re-run after pulling new skills — it adds new links, updates changed ones, and removes stale ones.

```bash
./bin/install-claude-skills-user
```

### Per-project

Installs each skill as an individual symlink in a project's `.claude/skills/` directory. Same re-run behavior as the global install.

```bash
./bin/install-claude-skills-project /path/to/your/project
```

Both scripts create per-skill symlinks rather than symlinking the entire directory, so `~/.claude/skills/` (or `.claude/skills/`) can also contain other skills without them being added to this repo.

## Creating new skills

Use the `create-skill` skill itself:

```
/create-skill A skill that runs the test suite and summarizes failures
```

## Part of kmac-claude-kit

This repo is one component of [kmac-claude-kit](https://github.com/kmacmcfarlane/kmac-claude-kit), a toolkit for building software with Claude Code. See that repo for how claude-skills, [claude-sandbox](https://github.com/kmacmcfarlane/claude-sandbox), and [claude-templates](https://github.com/kmacmcfarlane/claude-templates) fit together.

## License

This project is licensed under the [GPL-3.0](LICENSE).
