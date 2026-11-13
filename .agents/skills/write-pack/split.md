# Map the repos and split the work

A pack changes exactly one repo: `target.yaml` has one `repo`, and the run can
see no other. So before you write anything, find every repo the work touches,
and decide how many packs it takes.

## Map the repos

From the request and a first look at the code, list every repo the work
touches. For each one, say whether it:

- **changes**: the work edits code in it; or
- **is only read**: the change depends on it without editing it. Examples: an
  API the change calls, a shared schema, a library it builds on, the framework
  whose rules it must follow.

Follow the dependencies until you reach code the work doesn't depend on. A
request that names one repo often needs a second: a client for an API that
doesn't exist yet, a library that needs a new option first, a release before an
app can use it.

Show the person the map and ask them to confirm it.

Each repo on the map needs a local checkout. If one isn't checked out, or you
can't reach it, ask for it and wait. Draft nothing for a repo you can't read: a
pack written from guesses about code is worse than no pack.

- **A repo that changes** is read at the latest commit of its `base` branch. If
  its checkout is on another branch, behind, or has local changes, tell the
  person and ask. Don't switch, pull or reset it yourself.
- **A repo that is only read** is read at the version the changed repo uses,
  from its lockfile or version pin, not at its latest commit.

Each repo that changes must be one of the customer's target repos, agreed at
onboarding. `pack-ready` refuses any other.

## Split into packs

- **One pack per changed repo.**
- **Within one repo, suggest a split** when the request covers changes that
  don't depend on each other, so each pack can be reviewed and run on its own.
  The person decides.
- **Order.** Say which packs depend on which. A library change, and its
  release, come before the app that upgrades to it. An API comes before the
  client that calls it. Packs that don't depend on each other can run at the
  same time.
- **Follow the change to where it's seen.** Check where the person will see
  the outcome, from "How we'll know it worked" in the goal. A change to a
  library or service reaches a repo that uses it only when that repo upgrades:
  a release, then a version bump, and sometimes a step such as a data backfill
  or a migration. If the outcome is seen in the repo that uses it, the upgrade is
  part of the work. It goes in its own pack, or in that repo's pack, and that
  pack waits for the release.
- **A dependency that hangs on an open question.** Sometimes whether one pack
  needs another depends on a question nobody has answered yet. Don't settle it
  by assuming an answer. Split for the answer you suggest, and say in each
  pack's `## Open questions` what changes under the other answer, such as a
  pack that's no longer needed.

Agree the slugs with the person. Each slug is lowercase letters, digits and
`-`, and becomes the folder `packs/<customer>/<slug>/` and the branch
`pack/<customer>/<slug>`.

## Agree the seam once

Where one pack's work meets another's, agree the interface once, with the
person: endpoints and the shapes they take and return, method names and
arguments, option names and values, the version or tag the later pack depends
on. Write it into each pack's `spec.md` under a `## Interface` heading, word for
word the same in every pack, so the runs build the sides of the same thing. The
run's review checks the work against `spec.md`, so the interface has to be there,
not only in `refs/`.

Then, in each pack, write `refs/related-packs.md`:

```markdown
# Related packs

This run changes only `acme-co/portal`. The other repos below are changed by
their own packs.

| Order | Pack | Repo | What it does |
|---|---|---|---|
| 1 | `acme/export-api` | `acme-co/api` | Adds `GET /exports/:id`. |
| 2 | `acme/export-csv` (this pack) | `acme-co/portal` | Calls it from the export page. |

## The interface

<the agreed interface, word for word as under `## Interface` in each spec>
```

## What the run needs from repos it only reads

The run can't open another repo. Copy what it needs into this pack's `refs/`:
the part of the API, schema or rules it depends on, with the repo and the
version, tag or commit you copied it from at the top: the version the changed
repo uses. Keep it to what the work uses. If
the material is company documentation in the customer's docs repo, pin it with
`docs` in `target.yaml` instead.

## Packs that wait for another

A pack that depends on another can't have its plan written yet, because the plan
is drafted against real code. That code isn't there until the earlier pack's run
has opened its agent PR on the target repo, and that PR has been reviewed and
merged into `base`. Merging the earlier pack only starts its run.

- Write its `goal.md` and `spec.md` now, with the others, so the interface is
  settled in one sitting.
- Leave its plan until that agent PR has merged, and any release the pack needs
  is out. Keep the template's `plan.md` as it is until
  then. Picking up the pack later writes it.
- Under `## Open questions` in its `spec.md`, write what it waits for:

  ```markdown
  ## Open questions

  - Waits for the agent PR from `acme/export-api` to merge into `acme-co/api`,
    and for `acme-co/api` v2.3.0 to be released.
  ```

  `pack-ready` fails while that section has text. With the README's merge rules
  in place, the pack can't be merged, and its run can't start, before its time.
- Open its pull request as a draft, so reviewers can read it alongside the
  others.

Tell the person the order to merge the packs in, and what has to happen between
them, such as a release.
