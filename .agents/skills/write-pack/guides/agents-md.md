# Writing `AGENTS.md`

`AGENTS.md` holds the standing rules for working on this repo. The run's agent
reads it before it starts. Fill in the template's sections from the survey, and
delete every `FILL` comment as you go.

## The repo's own rules

List every guide you found for agents and contributors, nested ones included,
by path, and say they apply. For example:

```markdown
Follow these. They apply to every change:

- `CLAUDE.md`: the whole repo.
- `docs/testing.md`: how tests are written here.
- `CONTRIBUTING.md`: commit messages and code review.
```

Point at them. Don't copy them in: a copy goes stale the day someone edits the
original, and the run reads the repo as it is. The run's agent won't open a
nested guide unless this list names it. If the repo has none, write "None".

## Stack and layout

Language, framework and their versions. Where the main code, the tests and the
config live. Anything generated that mustn't be edited by hand, and the command
that regenerates it.

## How to run things

The exact commands, as the survey proved them: setup, the whole suite, a single
test, the linter. Write each command with no variables in front (`npm test`, not
`APP_ENV=test npm test`), and exactly as `permissions.allow` lists it: the run
can only run commands that match an entry there. Variables go in `env` in
`target.yaml`.

## Conventions

Point at an existing file that shows the house style for the kind of change this
pack makes, rather than describing the style. Add only what that file doesn't
show: a pattern to avoid, a helper to reuse.

## Definition of done

Keep the template's list, but adjust or drop any line about a tool the repo
doesn't have, such as a linter. Add a line only for something this repo needs
on top, such as a coverage floor CI enforces.
