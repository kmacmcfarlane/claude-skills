---
name: create-skill
description: Bootstrap a new Claude Code skill from a description. Use when the user wants to create a new skill/slash command.
disable-model-invocation: true
allowed-tools: Read, Write, Glob, Grep, Bash
argument-hint: <description of the skill to create>
---

# Create a new Claude Code skill

The user wants to create a new skill. Their description: $ARGUMENTS

## Process

1. **Derive a name** from the description. Use lowercase kebab-case (e.g. `run-tests`, `check-deps`). Keep it short (1-3 words).

2. **Decide on settings:**
   - `disable-model-invocation`: default `true` (user-invoked only). Set `false` if the skill should activate automatically based on conversation context.
   - `allowed-tools`: list only the tools the skill actually needs. Common sets:
     - Read-only research: `Read, Glob, Grep`
     - Code modification: `Read, Write, Edit, Glob, Grep, Bash`
     - Bash-heavy: `Bash, Read, Glob`
   - `argument-hint`: a brief hint shown in autocomplete (e.g. `<file-path>`, `<issue-number>`)
   - `context: fork` if the skill should run in an isolated subagent

3. **Write the SKILL.md** with this structure:

```yaml
---
name: <skill-name>
description: <1-2 sentence description of when to use this skill>
disable-model-invocation: <true|false>
allowed-tools: <comma-separated tool list>
argument-hint: <hint>
---

# <Title>

<Clear, concise instructions for Claude. Write in imperative mood.>

## Steps

<Numbered steps Claude should follow when the skill is invoked.>
```

4. **Create the skill directory and file:**
   - Determine whether to place it in the project (`.claude/skills/<name>/SKILL.md`) or in the user's skills repo. Ask the user if unclear.
   - Create the directory and write the SKILL.md file.
   - If the skill needs supporting files (templates, examples, reference docs), create those too.

5. **Report** what was created and how to invoke it (`/<skill-name>` or `/<skill-name> <args>`).

## Guidelines

- Keep SKILL.md under 5000 tokens. Move detailed reference material into supporting files.
- Instructions should be specific and actionable. Avoid vague guidance.
- Use `$ARGUMENTS` to reference user input. Use `$0`, `$1` etc. for positional args.
- Use `` !`command` `` syntax for dynamic preprocessing only when the skill genuinely needs runtime data injected before Claude sees the prompt.
- Don't over-engineer. A skill that does one thing well is better than one that tries to handle every edge case.
