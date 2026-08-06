# Development Skills Repository

A collection of reusable development skills for OpenCode that enforce coding standards, architectural patterns, and best practices across projects.

## Overview

This repository contains agent skills that provide consistent, enforceable guidelines for software development. Each skill is a self-contained set of rules, examples, and conventions that OpenCode can load and apply during development sessions.

## Available Skills

### [api-design](./api-design/)
Enforce API design standards for ASP.NET Core APIs including versioning strategy, status code declarations, TypedResults usage, ProblemDetails contracts, and response metadata.

**Coverage:**
- True REST resource modelling (properties vs sub-resources, PATCH over sub-resources)
- Versioning strategy (URL-prefix, not header or content-negotiation)
- TypedResults as the primary response documentation mechanism
- Standard status code catalogue (200, 201, 204, 400, 401, 403, 404, 408, 409, 422, 500)
- 408 for client abort/disconnect
- ProblemDetails for all 4xx and 5xx responses
- `[ProducesResponseType]` for supplemental descriptions
- Do-not-use guidance for `[SwaggerResponse]`
- URL casing (lowercase kebab-case) and enum formatting
- OpenAPI metadata conventions (`[ExcludeFromDescription]`, `[Tags]`, `[EndpointDescription]`, `[EndpointSummary]`)
- Strongly typed `IOptions<T>` over raw `IConfiguration`

**Use when:** Designing, documenting, or reviewing ASP.NET Core APIs — versioning strategy, status codes, TypedResults, or OpenAPI metadata.

---

### [csharp-development](./csharp-development/)
Enforce C# coding standards and best practices for .NET applications.

**Coverage:**
- Core principles (zero warnings, Boy Scout principle)
- Naming conventions (PascalCase/camelCase)
- Global usings and code organization
- Exception handling and control flow
- Fluent API formatting
- Namespace and folder structure alignment

**Use when:** Creating, modifying, or reviewing C# code in .NET projects.

---

### [testing-standards-and-skills](./testing-standards-and-skills/)
Enforce repository unit test standards for xUnit, Shouldly, naming, structure, and embedded test resources.

**Coverage:**
- xUnit test framework conventions
- Shouldly assertion library usage
- Test naming patterns (`MethodUnderTest_ExpectedBehavior_Condition`)
- Arrange/Act/Assert structure
- Test categorization (Unit/Integration traits)
- Version traits for multiple interface versions
- Project and folder structure
- Embedded resource management

**Use when:** Writing or reviewing unit and integration tests.

---

### [feature-sliced-architecture](./feature-sliced-architecture/)
Structure ASP.NET Core applications around vertical feature folders with isolated dependency injection, feature boundaries, and shared infrastructure modules.

**Coverage:**
- Vertical slice architecture principles
- Feature folder organization
- Dependency injection conventions
- Infrastructure module patterns
- Feature boundaries and coupling
- Configuration management
- Naming conventions for features and infrastructure

**Use when:** Building or refactoring ASP.NET Core applications with feature-based organization.

---

### [grill-me](./grill-me/)
Grill the user relentlessly about a plan, decision, or idea to stress-test their thinking.

**Coverage:**
- Design-tree modelling of decisions and their dependents
- Round-based questioning limited to the current "frontier" of unblocked decisions
- Numbered questions with a recommended answer for each
- Fact-finding delegated to sub-agents instead of asking the user
- Non-blocking exploration — unrelated frontier questions proceed while facts are gathered
- Session ends only when the full design tree has been visited

**Use when:** The user wants to stress-test a plan, decision, or idea, or uses any 'grill' trigger phrases.

---

## Installation

### Project-Level Installation

Copy the desired skill folders to your project:

```bash
# Create skills directory
mkdir -p .opencode/skills

# Copy skills you want to use
cp -r /path/to/this/repo/api-design .opencode/skills/
cp -r /path/to/this/repo/csharp-development .opencode/skills/
cp -r /path/to/this/repo/testing-standards-and-skills .opencode/skills/
cp -r /path/to/this/repo/feature-sliced-architecture .opencode/skills/
cp -r /path/to/this/repo/grill-me .opencode/skills/
```

### Global Installation

Install skills globally for use across all projects:

```bash
# Create global skills directory
mkdir -p ~/.config/opencode/skills

# Copy skills
cp -r /path/to/this/repo/api-design ~/.config/opencode/skills/
cp -r /path/to/this/repo/csharp-development ~/.config/opencode/skills/
cp -r /path/to/this/repo/testing-standards-and-skills ~/.config/opencode/skills/
cp -r /path/to/this/repo/feature-sliced-architecture ~/.config/opencode/skills/
cp -r /path/to/this/repo/grill-me ~/.config/opencode/skills/
```

### Alternative Paths

OpenCode also searches these locations:
- `.claude/skills/<name>/SKILL.md`
- `~/.claude/skills/<name>/SKILL.md`
- `.agents/skills/<name>/SKILL.md`
- `~/.agents/skills/<name>/SKILL.md`

## Usage

Once installed, OpenCode will automatically discover available skills. The agent can load skills when needed using the native skill tool.

Skills are loaded on-demand—the agent sees available skills and loads the full content when the task matches the skill's description.

## Skill Structure

Each skill follows this structure:

```
skill-name/
├── SKILL.md       # Main skill definition with YAML frontmatter
└── README.md      # Optional: Skill-specific documentation
```

### SKILL.md Format

```yaml
---
name: skill-name
description: Brief description for agent discovery
license: MIT | Proprietary
compatibility: opencode
metadata:
  key: value
---

## What I do
[Description of what the skill enforces]

## When to use me
[Guidance on when to apply this skill]

## [Sections with rules, examples, and guidance]
```

## Creating New Skills

To create a new skill:

1. Create a directory with a valid skill name (lowercase, alphanumeric, hyphens only)
2. Add a `SKILL.md` file with proper YAML frontmatter
3. Include clear sections: "What I do", "When to use me", and detailed rules
4. Provide concrete examples (good and bad)
5. Add enforcement guidelines

### Skill Name Requirements

- 1-64 characters
- Lowercase alphanumeric with single hyphen separators
- Cannot start or end with `-`
- No consecutive `--`
- Regex: `^[a-z0-9]+(-[a-z0-9]+)*$`

## Contributing

When adding or modifying skills:

1. Follow the established structure and formatting
2. Provide clear, actionable rules
3. Include concrete examples
4. Test the skill with OpenCode before committing
5. Update this README with new skills

## Philosophy

These skills embody several key principles:

- **Zero warnings**: Code must be clean and warning-free
- **Boy Scout principle**: Leave code better than you found it
- **Consistency**: Apply standards uniformly across the codebase
- **Clarity over cleverness**: Readable, maintainable code wins
- **Pragmatism**: Rules have clear rationale and exceptions when justified

## License

Individual skills may have different licenses. Check each skill's SKILL.md frontmatter for licensing information.

## Related Resources

- [OpenCode Documentation](https://opencode.ai/docs)
- [OpenCode Skills Guide](https://opencode.ai/docs/skills)
