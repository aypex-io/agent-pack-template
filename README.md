# agent-pack-template

A template for the repo where your company keeps its **packs** for the Agent
Run Platform.

A pack is the brief a coding agent works from: why the work matters, what done
looks like, how to work on your code, and the plan to follow. When a pack is
merged, the platform runs a coding agent against one of your repos and opens a
pull request with the result. A GitHub App named `aypex-agent-runs` does the
GitHub side: it checks packs, starts runs and posts status as comments. This
template is published by Aypex (`aypex-io`).

This repo holds `template/`, the empty pack to copy, and this guide. Make your
own packs repo from it and replace this README with your own notes.

## Before you start

Onboarding with the platform's operator settles these. None of them is written
in a pack.

- Your **customer slug**, such as `acme`. It is the first folder under `packs/`
  and the `customer` field in every pack.
- Your **packs repo**, your **docs repo** if you want to give agents company
  docs, and the **target repos** the agent may change.
- Your **operators team**, such as `acme-co/agent-operators`: the people who may
  re-run a merged pack or ask for a revision.
- What the agent's tools may reach over the network (`egress`).

## Lifecycle

1. **Install the App** on your GitHub org. Your operator sends you the install
   link, so don't start from the App's page before onboarding. Select your
   packs repo, your docs repo and the target repos. The App can only see what
   you select. A repo left out shows up like this:
   - the packs repo: no `pack-ready` check ever appears;
   - a target repo: the run fails after the merge, with a failure comment on the
     PR;
   - the docs repo: a pack that pins `docs` fails `pack-ready`, with a line
     saying the App can't read the docs repo.
2. **Make your packs repo** from this template. Its default branch must be
   `main`. Packs go in `packs/<customer>/<slug>/`.
3. **Create the operators team** named at onboarding, and two labels:
   `force:agent-run` on your packs repo, and `agent-revise` on each target repo.
   A label that doesn't exist can't be applied.
4. **Add a ruleset on `main`** (Settings, Rules, Rulesets), once your operator
   says your side of the cluster is ready. GitHub may not offer the App as the
   check's source until the check has run, so if it doesn't, open any PR first
   and wait for `pack-ready` to appear on it, within about 10 minutes. Add these
   rules:
   - "Require a pull request before merging", for your own approvals (for
     example one reviewer, and dismiss approvals when new commits are pushed);
   - "Require status checks to pass", with the check `pack-ready` and its source
     set to the App (`aypex-agent-runs`, integration ID `5571914`);
   - merge commits as the only allowed merge method. Turn off squash and
     rebase.

   Set its enforcement status to Active, and leave the bypass list empty. Without
   the required check, a pack that fails
   can still be merged, and merging starts the run. Rulesets on private repos
   need GitHub Pro, Team or Enterprise. Merging is the approval, so the ruleset
   decides who may start a run.
5. **Branch** `pack/<customer>/<slug>`, for example `pack/acme/export-csv`. The
   branch must be in this repo, not a fork. `<customer>` is your customer slug
   (see the `customer` field below). `<slug>` is lowercase letters, digits and
   `-`. A branch with any other slug starts nothing when it is merged.
6. **Write the five files.** Copy `template/` to `packs/<customer>/<slug>/` and
   fill in `goal.md`, `spec.md`, `AGENTS.md`, `target.yaml` and `plan.md`. See
   "The five files" and "Writing the plan".
7. **Open a PR** into `main`. Fix what `pack-ready` reports until it passes.
   Reviewers may push edits to any file in the pack, `plan.md` included.
8. **Merge with a merge commit**, as a person. **That starts the run.** Status
   arrives as a comment on the merged PR with a link to the agent's PR and a
   cost table. The agent's PR on your target repo is opened by the App and holds
   a summary, the agent's review, the cost table and a note that the platform
   re-ran the tests. If nothing appears, see "When a merge posts nothing".
9. **Re-run or revise.** Label the merged pack PR `force:agent-run` to re-run it
   from scratch: a new agent branch and a new agent PR. Label an agent PR
   `agent-revise` to have the agent address its review feedback: the PR must be
   open and have review feedback left since the agent's last commit, or the run
   fails and says so. Anyone with triage rights can apply either label, but only
   active members of the operators team get a run. A `force:agent-run` from
   anyone else is removed, with a comment saying so. An `agent-revise` from
   anyone else fails, with a comment. To change what was asked for, write a new
   pack. A merged pack never changes.

The agent never writes to your packs repo.

A pack PR adds one new folder, `packs/<customer>/<slug>/`, and changes nothing
else. Its branch is named for that folder.

## The five files

Every pack holds exactly these, plus an optional `refs/` folder for supporting
material such as API docs, designs and earlier decisions.

| File | Holds | What the check needs |
|---|---|---|
| `AGENTS.md` | Standing rules for working on your code: its own guides, stack, how to run tests, conventions, definition of done. | Present, with no `FILL` marker left. |
| `goal.md` | Why: the outcome, who it is for, how you will know it worked. | Present, no `FILL` marker. |
| `spec.md` | What: acceptance criteria, what happens when things go wrong, examples, out of scope. | A `## Acceptance criteria` section with at least one real list item (not the template's `Given …, when …, then …` or `…`), no `FILL` marker, and no text left under `## Open questions`. |
| `plan.md` | How: the approach, the changes file by file, the tests, the risks. The agent follows it. | Present, not empty, no larger than 1 MiB, no `FILL` marker. |
| `target.yaml` | Where and how: repo, runner, model, permissions, verify command, budget. | Valid against the field table below, with no `FILL` marker. |

All five must be plain UTF-8 text, regular files (no symlinks), with no NUL
byte. A `FILL` marker is the word `FILL` on any line, including inside a
comment. A pack can't carry a `.gitattributes` file anywhere in it.

## `target.yaml`

```yaml
contract: agent-run/v2
customer: acme
repo: acme-co/portal
base: main
runner: claude-code
model: claude-opus-5-5
stages: [implement, review]
permissions:
  allow: [read, edit, "bash:git *", "bash:npm test"]
  deny_paths: [".env*"]
verify: npm ci && npm test
env: {}
budget: { max_turns: 200, deadline: 3h, usd_cap: 40 }
egress: [registry.npmjs.org]
```

No field may be added that the table doesn't list.

| Field | Required | What it does, and its limits |
|---|---|---|
| `contract` | yes | Must be `agent-run/v2`. |
| `customer` | yes | Your customer slug: lowercase letters, digits and `-`, 3 to 32 characters, starting with a letter and not ending in `-`. It must equal the `<customer>` folder the pack sits in. |
| `repo` | yes | The repo the agent changes, as `owner/name`. It must be one of your target repos, in your org. It can't be your packs repo or your docs repo. |
| `base` | yes | The branch the agent's branch is cut from, and the branch its PR targets. |
| `runner` | yes | The runner image, such as `claude-code`: lowercase letters, digits and `-`. The runner images your setup offers are settled at onboarding. |
| `model` | yes | The model ID, in your model provider's own naming. The runner checks it. The models your setup offers are settled at onboarding. |
| `stages` | yes | The agent stages, in order: `[implement]` or `[implement, review]`. `[review]` alone is invalid; nothing repeats. `verify` always runs after them. |
| `permissions.allow` | yes | What the agent may do; at least one entry. Anything not listed is denied. `read` reads files. `edit` changes files. `web` allows web search and fetch. `ci` allows changes under `.github/workflows/` and `.github/actions/`; without it, a run that changes those paths fails before anything is pushed. `bash:<pattern>` allows shell commands matching the glob, such as `bash:npm test` or `bash:git *`. A command that starts with a variable assignment (`FOO=1 npm test`) doesn't match a pattern, so set variables in `env`. |
| `permissions.deny_paths` | no | Globs, relative to the repo root, that the agent can't read or write even when `read` and `edit` are allowed, such as `.env*`. |
| `verify` | yes | A shell command the platform runs in a clean container against the agent's commits. It must exit 0 before a PR is opened. The agent's own claim that tests pass isn't trusted. |
| `env` | no | Variables set for the agent's tools and for `verify`, such as `RAILS_ENV: test`. Names start with a capital letter and hold only capital letters, digits and `_` (`^[A-Z][A-Z0-9_]*$`); values are strings or integers. The names the platform sets are refused: `PATH`, `HOME`, `SHELL`, `TMPDIR`, `BASH_ENV`, `ENV`, `NODE_OPTIONS`, `DISABLE_AUTOUPDATER`, the variables the platform sets for `services` (listed there), and any name starting `AGENT_`, `MODEL_`, `ANTHROPIC_`, `CLAUDE_`, `GIT_` or `LD_`. |
| `budget.max_turns` | yes | Most turns the agent may take: a whole number from 1 to 2000. |
| `budget.deadline` | yes | Wall-clock limit for the whole run, a whole number then `m` or `h`: `90m`, `3h`. |
| `budget.usd_cap` | yes | Spending cap in US dollars; above 0. |
| `egress` | no | Extra hosts the agent's tools may reach, such as package registries: lowercase letters, digits, `.` and `-`. Each must be on the list agreed at onboarding. |
| `toolchain` | no | What the agent and `verify` run on: `node` (the default, Node 22) or `ruby` (Ruby 4.0.1, with Node alongside). |
| `services` | no | Backing services started for the whole run and reachable from the agent's tools and `verify`. `postgres` is PostgreSQL 18.6, empty at the start of the run, at `DATABASE_HOST` / `PGHOST`. The platform sets `DATABASE_HOST`, `DATABASE_PORT`, `DATABASE_USERNAME`, `DATABASE_PASSWORD`, `PGHOST`, `PGPORT`, `PGUSER` and `PGPASSWORD`, so tests can connect, and a pack's `env` can't set them. It doesn't set `DATABASE_URL`. The user can create databases. Ask your operator which others exist. |
| `docs` | no | Company docs pinned for this run, readable at `/docs`. See below. |

### `docs`

```yaml
docs:
  ref: 0123456789abcdef0123456789abcdef01234567
  paths: [handbook/engineering.md, "style/*.md"]
```

Both keys are required if `docs` is there. The agent reads the files at
`/docs`, read-only.

- `ref` is a full 40-character commit SHA, in lowercase hex, on the **default
  branch** of your docs repo. The docs repo isn't named in the pack: it is the
  one agreed at onboarding, and your customer must have one.
- `paths` is a list of at least one path or glob inside the docs repo.
  Paths are relative: no leading `/`, no empty segment, no `..`. Use only letters, digits, `_`, `.`, `-`, `/`, `*` and `?`. A path can't start with `.` or `-`, and can't hold spaces or `[ ]`. `*` and `?`
  stay within one folder, so `style/*.md` is one level. `**` crosses folders, so
  `style/**/*.md` is the whole tree. Every entry must match at least one file.
- Only regular files count. Symlinks and submodules don't match.
- Limits: 1 MiB for each file and 5 MiB for all matched files together.

Pin the SHA, not a branch, so a run always sees the docs as they were when the
pack was reviewed.

## The `pack-ready` check

`pack-ready` is the App's own check run on your PR. It isn't a GitHub Actions
job, so there is no workflow file to add and nothing appears in the Actions tab.

It is also the validator. **There is no local validator: push the branch to run
the check.** Each failure is one line saying what to fix, and the check runs
again on the next push.

It runs on every PR into your default branch. A PR that changes no pack passes
("Ready: no pack changed"), unless its branch is named `pack/…` or it touches
anything under `packs/`. On a pack PR it checks:

- the branch is `pack/<customer>/<slug>` and matches the folder, and the PR adds
  one new pack folder and nothing else;
- the five files are there, each plain text with no NUL byte, with no symlink
  and no `.gitattributes` anywhere in the pack;
- no `FILL` marker is left, `spec.md` has real acceptance criteria and no open
  questions, and `plan.md` isn't empty or over 1 MiB;
- `target.yaml` follows the field table, and its `customer` matches the folder;
- the things your setup would refuse after the merge: a customer the platform
  doesn't serve, a `repo` outside your targets or org, `egress` hosts not on
  your list, and a `docs` pin that doesn't resolve (including the size limits).

PRs must come from branches in your packs repo. A PR from a fork always fails.

**When it starts.** On a push to the PR, when the PR's base is changed onto the
default branch, and on **Re-run**.

- Use Re-run on the `pack-ready` check itself. "Re-run all checks" isn't
  handled.
- Re-run does nothing on a fork PR. A push, or closing and reopening the PR,
  checks it again.
- If the check is missing, it appears within about 10 minutes without a push.
- If the check says "pack-ready couldn't run", the check failed, not your pack.
  Press **Re-run** on it. A check that has finished isn't retried on its own. If
  it fails again, ask your operator.

## Writing the plan

Write the plan before the PR, in a local agent session that can read your target
repo. You don't need the platform to do it.

1. Write `goal.md`, `spec.md`, `AGENTS.md` and `target.yaml` first.
2. Point the session at your repo, `spec.md` and the empty `plan.md`. Ask it to
   read the code and draft the plan against the acceptance criteria. The template's
   `plan.md` has the sections: Approach, Changes (file by file), Tests (one for
   every acceptance criterion, saying what it asserts and that it fails if the
   code it covers is removed), Risks and open questions, and Out of scope.
3. Engineering and product edit the draft. Check the file paths and test names
   against the repo. Move anything uncertain into "Risks and open questions".
4. Delete every `FILL` comment as you fill it in. The check refuses a leftover
   marker, and refuses an empty `plan.md` or one over 1 MiB.

Merging approves the plan as written, and the agent follows it. Review it as you
would a design.

## When a merge posts nothing

Some merges start nothing and post nothing, because the event is dropped before
a run begins:

- the PR isn't from a `pack/<customer>/<slug>` branch in your packs repo (a fork
  included), or `<customer>` isn't a customer registered for this repo;
- the PR wasn't merged into `main`;
- **the person who merged has a bot login** (a GitHub App, whose login ends in
  `[bot]`) **or a login with an underscore**, as GitHub Enterprise Managed Users
  have (`name_acme`). The run starts nothing and posts nothing. This is a known
  limit. Merge as a person with an ordinary login. The same goes for a
  `force:agent-run` or `agent-revise` label.

Other merges start a run that is refused, with a comment on the PR saying why:

- the merge wasn't a merge commit (squash or rebase);
- the merge changed more than the one new pack folder, or the pack no longer
  passes the check;
- a problem with the target repo, the egress hosts or the `docs` pin.

If the cluster is down when you merge, the event may be lost. Wait a few
minutes. If no status comment has appeared and none of the above fits, ask your
operator to re-run the merged pack with the `force:agent-run` label.

## Re-running

`force:agent-run` works only on a merged pack PR. On an open PR it does nothing,
and stays on the PR doing nothing after the merge. To re-run, remove it and add
it again on the merged PR. It starts a whole new attempt, with a new agent
branch and a new agent PR. Earlier agent PRs are left alone.

If a merged pack is right but the platform wasn't ready for it, fix what was
missing and re-run. If the pack itself is wrong, write a new pack with a new
slug.
