# CLAUDE.md

# Mineradio Linux

This project is a Linux version of Mineradio, developed based on the original open-source Mineradio project.

The goal is to complete a modern, maintainable, and high-quality Linux desktop music player while preserving the original project's architecture whenever possible.

---

# Before Starting Any Work

Before beginning any development task, Claude **MUST** read the documentation inside the `docs/` directory and AGENTS.md.

The following documents should always be reviewed first:

- docs/README.md
- docs/roadmap/ROADMAP.md
- docs/design/DESIGN.md
- Any other documentation related to the current task

Do **NOT** start coding before understanding the current project status and design.

---

# Development Workflow

Development must strictly follow the design document.

## Rule 1

Implement **only one small feature/module at a time**.

Avoid implementing multiple unrelated features in a single iteration.

## Rule 2

Follow the implementation order defined in `docs/DESIGN.md`.

Never skip unfinished sections.

## Rule 3

After completing each feature:

- Verify that the project still builds successfully.
- Fix compiler warnings whenever possible.
- Ensure existing functionality is not broken.

---

# Version Management

Every completed development milestone must create a Git tag.

The tag should represent a small completed version instead of a large release.

Example:

v0.1.0
v0.1.1
v0.2.0

For every tag:

- Update CHANGELOG.md
- Record:
  - Added features
  - Fixed bugs
  - Refactoring
  - Known issues (if any)

Every version should have clear release notes.

---

# Coding Standards

## General Principles

Write code that is:

- Readable
- Maintainable
- Modular
- Reusable

Avoid unnecessary complexity.

Prefer simple and elegant solutions.

---

## Naming

Use descriptive English names.

Good:

```
currentSong
playListModel
musicPlayer
```

Bad:

```
tmp
aaa
test1
```

---

## Functions

Each function should have **one responsibility**.

Prefer functions shorter than 100 lines.

Split large functions into smaller ones.

---

## Classes

Each class should represent one responsibility.

Avoid "God classes".

Use encapsulation whenever possible.

---

## Comments

Only write comments when they explain **why**, not **what**.

Bad:

```cpp
// Increase i
i++;
```

Good:

```cpp
// Prevent duplicated refresh events.
```

---

## Formatting

Keep formatting consistent.

Use the project's existing coding style.

Do not introduce formatting changes unrelated to the task.

---

# Git Commit Rules

Commit messages should follow:

```
type(scope): summary
```

Examples:

```
feat(player): add lyric parser

fix(ui): resolve playlist refresh bug

refactor(core): simplify audio manager

docs: update development workflow
```

Commit types:

- feat
- fix
- refactor
- docs
- style
- test
- chore

---

# Documentation

Whenever a new feature is completed:

Update documentation if necessary.

Possible files:

- DESIGN.md
- ROADMAP.md
- CHANGELOG.md
- README.md

Documentation is considered part of the task.

---

# Project Structure

Respect the existing directory structure.

Do not move files unless necessary.

Avoid creating unnecessary directories.

---

# Dependencies

Prefer existing libraries already used by the project.

Do not introduce new dependencies unless they provide significant value.

If a new dependency is required:

- Explain why.
- Minimize its impact.
- Document it.

---

# Error Handling

Never silently ignore errors.

Provide meaningful log messages.

Fail gracefully whenever possible.

---

# Performance

Avoid premature optimization.

However:

- Avoid unnecessary memory allocations.
- Avoid duplicate computations.
- Prefer efficient algorithms when complexity is obvious.

---

# Security

Never expose:

- API keys
- Tokens
- Passwords
- Personal information

Never hardcode secrets into source code.

---

# Testing

After every completed feature:

- Build the project.
- Run existing tests if available.
- Ensure no obvious regressions.

---

# Development Philosophy

The priority order is:

1. Correctness
2. Maintainability
3. Readability
4. Performance

Never sacrifice maintainability for minor optimizations.

When uncertain, choose the simpler implementation.

---

# AI Assistant Rules

Claude should never:

- Rewrite unrelated files.
- Perform large-scale refactoring without request.
- Change public APIs unless required.
- Introduce breaking changes without explanation.
- Guess project architecture.
- Skip reading documentation.

Claude should always:

- Think before coding.
- Explain major design decisions.
- Keep changes minimal.
- Preserve backward compatibility whenever possible.
- Update documentation after implementation.
- Show users in Chinese
- Never stop immediately after writing code.
- Always continue with verification, testing, self-review, documentation updates, and version recording before ending a task.
- Proactively identify and fix obvious issues discovered during verification without waiting for user instructions.
  
# Context Management

When the conversation is summarized or compacted, preserve the following information:

## Always Preserve

- Current development stage
- Current roadmap progress
- Unfinished TODO items
- Architecture decisions
- Design rationale
- Recently modified files
- Pending bugs
- Git version and latest tag

## May Discard

- Intermediate reasoning
- Repeated discussions
- Temporary debugging logs
- Obsolete implementation attempts

The summary should allow development to continue seamlessly after context compaction.

# Definition of Done

A feature is **NOT** considered complete immediately after implementation.

For every completed feature, Claude **MUST** automatically perform the following workflow before considering the task finished.

## 1. Build & Verification

- Build the entire project successfully.
- Resolve all build errors.
- Resolve compiler warnings whenever possible.
- Run all available tests.
- Verify that existing functionality has not been broken.

## 2. Code Review

If the official Claude Code Review Skill is available, Claude MUST use it to review the implementation.

Otherwise, Claude must perform a manual review using the same review standards.

The review should cover:

- Logic correctness
- Architecture consistency
- Error handling
- Resource management
- Memory safety
- Thread safety
- Performance
- Readability
- Maintainability
- Coding style consistency

Any issues discovered during review must be fixed before proceeding.

## 3. Validation Loop

After making any fixes, Claude must:

- Rebuild the project.
- Run tests again.
- Verify that previous issues have been resolved.

Repeat this verification and repair cycle until no obvious issues remain.

## 4. Documentation

If the feature changes project behavior, architecture, or progress, update the relevant documentation, including:

- CHANGELOG.md
- ROADMAP.md
- DESIGN.md
- README.md

Documentation updates are considered part of the implementation, not an optional step.

## 5. Version Control

After the feature has been fully verified:

- Create a Git commit following the project's commit convention.
- If the feature represents a completed milestone, create a Git tag.
- Update CHANGELOG.md with:
  - Added features
  - Bug fixes
  - Refactoring
  - Known issues (if any)

## Completion Criteria

Claude must **NOT** consider a task finished simply because the code compiles.

A task is considered complete **only after**:

- The implementation is finished.
- The project builds successfully.
- Tests pass successfully.
- Obvious issues have been reviewed and fixed.
- Documentation has been updated.
- Git history has been recorded.
- The corresponding version tag has been created (when applicable).

Implementation alone is **NOT** task completion.

## Mandatory Code Review

Every completed feature MUST be reviewed.

Priority:

1. Official Claude Code Review Skill
2. Manual review (if the Skill is unavailable)

Code review is a mandatory step of the development workflow and may not be skipped.