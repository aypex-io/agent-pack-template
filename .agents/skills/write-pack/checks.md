# Check the pack and open the PRs

## Check before you push

Read the current pack rules first (see "The pack rules" in
[the skill](SKILL.md)). Then check each pack against them. The ones that most
often fail:

- **Branch:** `pack/<customer>/<slug>`, matching the folder
  `packs/<customer>/<slug>/`, in this repo, not a fork.
- **One folder:** the PR adds one new pack folder and changes nothing else.
- **Files:** the five files are there, each plain UTF-8 text, with no symlink and
  no `.gitattributes` anywhere in the pack.
- **`FILL`:** the word `FILL` appears nowhere in the pack, comments included.
- **`spec.md`:** `## Acceptance criteria` has at least one real criterion, not
  the template's placeholder. `## Open questions` is empty, unless the pack is
  meant to wait, as a dependent pack does. A pack whose product questions are
  still open can be pushed so reviewers can answer them: open it as a draft,
  and list the questions in the PR description.
- **`plan.md`:** not empty, and no larger than 1 MiB.
- **`target.yaml`:** only the fields the README's table lists, each within its
  limits, and `customer` equal to the folder name.

If this packs repo's own README names a local validator, run it too.

## Open the PRs

Only if the person asks. One pack, one branch, one PR:

1. Make the branch `pack/<customer>/<slug>` from the latest `main`.
2. Commit only that pack's folder.
3. Push the branch, and open a PR into `main`. Open a pack that waits for
   another as a **draft**.
4. Wait for the `pack-ready` check. It appears within about 10 minutes. It
   isn't a GitHub Actions job, so it doesn't show in the Actions tab. Fix what
   it reports, and push again. Each push checks the pack again.

## Tell the person

- **Merging starts the run.** It must be a merge commit, made by a person. A
  squash or rebase merge is refused, and a merge by a bot account starts
  nothing.
- **The order** to merge the packs in, and what has to happen between them,
  such as a release.
- **Review the plan like a design.** Merging approves it as written.

Never merge a pack PR yourself, and never add the `force:agent-run` or
`agent-revise` labels.
