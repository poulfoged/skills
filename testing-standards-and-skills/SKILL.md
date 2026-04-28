---
name: testing-standards-and-skills
description: Enforce repository unit test standards for xUnit, Shouldly, naming, structure, and embedded test resources.
license: Proprietary
compatibility: opencode
metadata:
  audience: contributors
  focus: unit-testing
---

## What I do
- Enforce mandatory unit test conventions in this repository.
- Review new and modified tests for style, structure, and organization.
- Provide compliant examples and direct fixes when tests violate standards.

## When to use me
Use this skill whenever you create, modify, or review unit tests in this repository.

## Required testing framework
- Use `xUnit`.
- Use `[Fact]` for test cases.
- Use `async Task` for async behavior when needed.
- Use `Shouldly` for assertions.

## Test naming convention
Pattern:

```text
<MethodUnderTest>_<ExpectedBehavior>_<Condition>
```

Rules:
- Use PascalCase for each segment.
- Keep names explicit and descriptive.
- Each test describes one behavior.
- Avoid conjunctions like `And` or `Or` unless unavoidable.
- Do not encode implementation details.

Example:

```csharp
Has_been_posted_returns_false_when_not_posted
```

## Required test structure
Every test must contain exactly three sections in this order, each prefixed with four slashes:

```text
//// Arrange
//// Act
//// Assert
```

No additional sections, comments, or reordering.

## Section rules
Arrange:
- Set up only what is required for behavior under test.
- Prefer real dependencies through DI when reasonable.
- Avoid unnecessary mocks, data, and helper logic.

Act:
- Perform one action.
- Keep the action minimal (prefer one line).
- Capture the result in a clearly named variable.

Assert:
- Assert one outcome.
- Use Shouldly assertions only.
- Avoid logic, branching, or transformations inside assertions.

## Canonical example

```csharp
[Fact]
public async Task Has_been_posted_returns_false_when_not_posted()
{
    //// Arrange
    var tracker = _serviceProvider.GetRequiredService<ISocialPostTracker>();
    var puzzleId = $"test-puzzle-{Guid.NewGuid()}";

    //// Act
    var result = await tracker.HasBeenPostedAsync(puzzleId, SocialPlatform.Bluesky);

    //// Assert
    result.ShouldBeFalse();
}
```

## Project and folder structure
Mirror production paths inside test projects.

Production file:

```text
/Features/Images/IcoLoader.cs
```

Test file:

```text
/Features/Images/IcoLoaderTests.cs
```

Guidelines:
- Folder names match exactly.
- Test file name is production type name plus `Tests`.
- One test class per production type.
- Do not flatten or reorganize test project folders.

## Test files and embedded resources
For PDFs, images, JSON, and similar test files:
- Store files under `/TestFiles` in the test project.
- Mark files as `Embedded Resource`.
- Do not read test files from disk at runtime.
- Access resources through the test assembly.

Resource pointer pattern:
- Create a sealed empty marker class named `ResourcePointer`.
- Place it in the same `/TestFiles` folder.
- Use it to locate assembly and namespace.

Required loading pattern:

```csharp
typeof(ResourcePointer)
    .Assembly
    .GetManifestResourceStream(
        typeof(ResourcePointer),
        "diploma-with-fields.pdf");
```

Do not:
- Hard-code assembly names.
- Use relative or absolute file paths.
- Duplicate resource-loading helpers unless needed.

## General principles
- Tests read like executable specifications.
- One behavior per test.
- Clarity over cleverness.
- Failures communicate intent clearly.
- Keep Act and Assert sections minimal.

## Enforcement
- Use PR checklists that reference this skill.
- Enforce in code review.
- Optionally add templates or Roslyn analyzers.
