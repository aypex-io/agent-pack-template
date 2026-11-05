# Working on this repository

Standing rules for any agent working on this customer's code. The platform adds
its own rules on top; these are about this codebase.

## The repo's own rules

<!-- FILL: list the repo's own guides for agents and contributors, such as
     CLAUDE.md, AGENTS.md, CONTRIBUTING.md or test/AGENTS.md, and say they apply.
     Point at them rather than copying them, so they can't go stale. Not every
     agent reads them on its own, and none reads a nested one unprompted.
     Write "None" if the repo has none. -->

## Stack and layout

<!-- FILL: language, framework and versions; where the main code, tests and
     config live; anything generated that must not be hand-edited. -->

## How to run things

<!-- FILL: the exact commands. Setup, the test suite, a single test, the linter. -->

## Conventions

<!-- FILL: patterns to follow and ones to avoid. Point at an existing file that
     shows the house style rather than describing it. -->

## Definition of done

- The acceptance criteria in `spec.md` are met, each with a test where one is possible.
- Each new test fails when the code it covers is removed. For tests that guard
  data or access, such as tenant scoping or permissions, check this by breaking
  the code and running them.
- The full test suite and the linter pass.
- No unrelated changes, reformatting or dependency bumps.
- Commit messages say what changed and why.
