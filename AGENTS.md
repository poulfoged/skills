# AGENTS.md

## Repo purpose

A collection of OpenCode skills — reusable instruction files loaded on-demand by the agent skill tool. No build system, tests, or CI.

## Structure

```
<skill-name>/
  SKILL.md    ← required; YAML frontmatter + skill content
```

Each skill is one directory containing exactly `SKILL.md`. No other files are required.

## Skill name rules

- Lowercase alphanumeric with single hyphens: `^[a-z0-9]+(-[a-z0-9]+)*$`
- 1–64 characters, cannot start or end with `-`, no `--`

## SKILL.md frontmatter (required fields)

```yaml
---
name: skill-name          # must match directory name
description: ...          # used by agent for discovery — keep it precise
license: MIT | Proprietary
compatibility: opencode
---
```

## Installation paths OpenCode searches

Skills must be placed in one of these locations to be auto-discovered:

- `.opencode/skills/<name>/SKILL.md` (project-local)
- `~/.config/opencode/skills/<name>/SKILL.md` (global)
- `.claude/skills/<name>/SKILL.md`
- `~/.claude/skills/<name>/SKILL.md`
- `.agents/skills/<name>/SKILL.md`
- `~/.agents/skills/<name>/SKILL.md`

## Agent behaviour

- Give running progress updates on any task that touches more than one file or takes more than a few steps.
- Don't silently work for a long time before responding — surface what you're doing as you go.

## When modifying skills

- The `description` field drives agent discovery — it must accurately describe when to use the skill.
- Keep rules actionable and concrete; avoid generic advice.
- Update `README.md` at repo root when adding or removing a skill.
- Test the skill with OpenCode before committing (load it via the skill tool and verify it activates correctly).

## Available skills

| Skill | When to load |
|---|---|
| `api-design` | Designing, documenting, or reviewing ASP.NET Core APIs — versioning strategy, status codes, TypedResults, or OpenAPI metadata |
| `csharp-development` | Creating, modifying, or reviewing C# / .NET code |
| `testing-standards-and-skills` | Writing or reviewing xUnit tests |
| `feature-sliced-architecture` | Building or refactoring ASP.NET Core apps with feature folders |
| `grill-me` | Stress-testing a plan, decision, or idea, or on any 'grill' trigger phrase |
