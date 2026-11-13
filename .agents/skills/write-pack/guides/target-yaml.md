# Writing `target.yaml`

`target.yaml` says where the run works and what it may do. Start from the
template's copy, fill every field from the survey, and delete every `FILL`
marker. The field table in the template's README has the exact limits for each
field (see "The pack rules" in [the skill](../SKILL.md)). This guide says where
each value comes from. If you're a separate survey agent, wherever this guide
says to ask or tell the person, put it under **For the person** in your report
instead.

## Where each value comes from

- **`customer`**: the folder the pack sits in, `packs/<customer>/`.
- **`repo`**: the repo this pack changes, as `owner/name`. It must be one of the
  customer's target repos, agreed at onboarding.
- **`base`**: the branch the work starts from and the PR goes into. Usually the
  default branch. Ask if the repo works from another.
- **`runner` and `model`**: copy them from the newest pack already in `packs/`.
  They were agreed at onboarding, and earlier packs carry them. For the
  customer's first pack, ask the person, and don't keep the template's values
  without asking. Tell them that a wrong `runner` or `model` isn't caught until
  after the merge, and a merged pack can't be fixed, only replaced by a new one.
- **`egress`**: the hosts the survey found under **Registries**, and no others.
  Earlier packs show which hosts are on the customer's agreed list. A host no
  earlier pack uses may not be agreed, so ask the person. `pack-ready` refuses a
  host that isn't on the list, and a missing host fails `verify` after the
  merge.
- **`stages`**: `[implement, review]`. The review stage is a second, fresh
  agent checking the work against `spec.md`. Drop it only if the person asks.
- **`permissions.allow`**:
  - `read` and `edit`;
  - `"bash:git *"`, so the agent can commit;
  - one `bash:` entry for each command the agent must run: the test suite, a
    single test, the linter, setup. An entry is a glob matched against the
    whole command: `"bash:npm test"` allows only `npm test` itself. To allow
    arguments too, add the command ending in ` *`, such as `"bash:npm test *"`.
    A command with a variable in front never matches;
  - `ci` only if the change has to touch `.github/workflows/` or
    `.github/actions/`. Without it, a run that touches them is stopped. Granting
    it lets the agent's changes run in CI with the repo's secrets, so add it
    only when the spec needs a CI change;
  - `web` only if the work needs the agent to search or fetch from the web.
- **`permissions.deny_paths`**: `.env*`, plus the repo's own secret files, such
  as `secrets/**`, `*.pem` or `*.key`.
- **`verify`**: what CI runs before a merge, setup included, as one shell
  command. For example `npm ci && npm test`. It runs again in a clean Linux
  container after the agent finishes, and no PR opens unless it passes.
- **`env`**: the variables the tests need. Never put them in front of a command.
- **`toolchain`**: one of the toolchains the README lists, each at a fixed
  version. Pick the one the repo needs, and compare its version with the repo's
  version files. If none fits the repo's language or version, stop and tell the
  person: the run can't work on this repo yet.
- **`services`**: the backing services the tests need, from those the README
  lists. For any other service, ask the operator what exists.
- **`budget`**: size it to the change. The template's 200 turns, `3h` and `$40`
  suit a large change. A small one needs far less, such as 60 turns, `60m` and
  `$5`. The run stops at whichever limit it reaches first, and the spend is
  charged to the customer's own model account.
- **`docs`**: only if the agent needs company docs from the customer's docs
  repo. Pin a full 40-character commit SHA on the docs repo's default branch,
  and list only the paths the work needs.

## Before the first pack for a repo

If this is the first pack for this repo, or the pack changes `verify` from what
earlier packs used, tell the person to ask their operator to test `verify` in
the cluster before they merge. A `verify` that passes here but fails in the
clean Linux container is otherwise found only after the merge. By then the run
has spent its budget and opened no PR, and the fix is a new pack. The test uses
no model and writes nothing to GitHub.
