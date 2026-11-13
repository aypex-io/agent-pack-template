# Survey a repo

Find out how to work on one repo, and draft its pack's `AGENTS.md` and
`target.yaml`. The repo answers most of it. What only the person can answer,
the survey collects for the main session.

Run this as a separate agent per changed repo if your coding agent can start
one, on your cheapest capable model, while the main session works on the goal.
Otherwise the main session does it, in order.

If you're a separate agent, don't ask the person anything: you can't reach them.
Wherever this file or the guides say to ask or tell the person, put it under
**For the person** in your report, with the answer you'd suggest.

## What the survey is given

- the path of the repo's local checkout;
- a sentence or two on what the work is;
- the pack folder to write into, `packs/<customer>/<slug>/`;
- the packs repo, to read earlier packs from.

## What to read

- **The repo's own guides** for agents and contributors: `CLAUDE.md`,
  `AGENTS.md`, `CONTRIBUTING.md`, and nested ones in subfolders, such as
  `tests/AGENTS.md` or `api/CLAUDE.md`. Search the whole tree, not just the root. The run's agent
  won't open a nested guide unless the pack points at it.
- **CI**, for example `.github/workflows/`. What CI runs before a merge is the
  repo's own definition of passing, and the best candidate for `verify`.
- **The stack:** language and framework versions (lockfiles and version files such as
  `.tool-versions`, `.nvmrc` or the `engines` field in `package.json`), where the main code, tests and config
  live, and anything generated that mustn't be edited by hand.
- **The commands:** setup, the whole test suite, a single test, the linter.
  Prefer what CI and the guides use over what you'd guess.
- **Services** the tests need, such as a database, and how they find them.
- **Variables** the tests need set, such as `APP_ENV=test`. Find them in CI,
  the guides or an example file such as `.env.example`. Don't open secret files
  (`.env*`, credentials, keys), and never copy a secret value into the pack. If
  the tests need a secret, say so under **For the person**.
- **Registries and hosts** that setup downloads from, such as
  `registry.npmjs.org`. `egress` in `target.yaml` is built from this list.

## Prove the commands run

Run the test command in the narrowest form that still proves it works: one test
file, or the linter on one file.

- Use only the repo's own setup and test commands, the way a developer on the
  project would. Don't install system software. Don't start services.
- Read a setup script before you run it. Don't run one that installs system
  software, starts services or resets a database: put it under
  **For the person** instead.
- Run `git status --porcelain` before and after. If running the commands changed
  a tracked file, such as a lockfile or a schema dump, or left new untracked
  files, report it under **Tracked files changed**. Don't restore it yourself if you're a separate
  agent: the main session asks the person. If you're the main session, ask, and
  restore it only with their OK.
- If a command can't run on this machine (a service isn't running, a tool is
  missing), say so and why. Don't work around it.
- Don't copy the repo or install other versions of its dependencies to test
  against. If the work needs testing across versions, say so in the findings,
  and the plan covers it.

## What the run will see differently

The run isn't this machine. Note each difference that matters for this repo:

- It runs on **Linux**, in a clean container. Anything that depends on macOS, a
  local path or a file outside the repo won't be there.
- It runs on **one of the README's toolchains**, at that toolchain's version.
  Compare it with the repo's version files, and say if they differ.
- It reaches only the **`egress` hosts** the pack lists. Setup that downloads
  from anywhere else fails.
- **Services start empty.** A database has no schema and no data until the
  tests create them.
- **`verify` starts from a clean checkout**: no installed dependencies, no build
  output, no local config files that git ignores.

## Write the drafts

Write `AGENTS.md` by [guides/agents-md.md](guides/agents-md.md), and
`target.yaml` by [guides/target-yaml.md](guides/target-yaml.md), into the
pack folder.

## Report back

Return this note to the main session, under exactly these headings. Write
"None" under any heading with nothing to say.

```markdown
## Commands proven
<each command you ran, and whether it passed; the narrowest test command that works>

## CI's gate
<what CI runs before a merge, and the file it's in>

## Services
<what the tests need, and how they connect>

## Variables
<variables the tests need set>

## Registries
<hosts setup downloads from>

## What the run will see differently
<the differences above that matter for this repo>

## Couldn't find
<anything you looked for and didn't find, or couldn't run, and why>

## Tracked files changed
<tracked files the commands changed, and new untracked files, or "None">

## For the person
<questions, each with the answer you'd suggest, and warnings they must see>
```
