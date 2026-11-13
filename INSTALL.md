# Install Aypex Agent Runs

Aypex Agent Runs turns a reviewed brief into a pull request. You write a **pack**
that says what you want and how to work on your code. When someone merges it, a
coding agent does the work and opens a pull request on your repo for you to review.

A GitHub App, [`aypex-agent-runs`](https://github.com/apps/aypex-agent-runs), does the
GitHub side. Installing the App isn't enough on its own: nothing runs until Aypex
has registered your organization.

Setup takes about an hour of your time, plus waiting for Aypex in step 6.

## Before you start

You need:

- **A GitHub organization you're an owner of.** You'll install an App, create a team
  and add a ruleset.
- **GitHub Pro, Team or Enterprise,** if your packs repo is private. GitHub only
  offers rulesets on private repos on those plans.
- **Ordinary GitHub logins for the people who merge.** Aypex Agent Runs can't yet
  start a run for a login with an underscore, and Enterprise Managed Users all have
  one (`name_acme`). A merge by one of those starts nothing and posts nothing. If your
  organization uses Enterprise Managed Users, tell Aypex before you go further.
- **An Anthropic account you can create API keys in.** Step 5 covers the key.

## Step 1: Agree the details with Aypex

Aypex registers these for you, so settle them first.

| What | Example | Notes |
|---|---|---|
| Customer slug | `acme` | Lowercase letters, digits and `-`, 3 to 32 characters. It names your pack folders. |
| Packs repo | `acme-co/packs` | You create it in step 2. Its default branch must be `main`. |
| Target repos | `acme-co/portal` | The only repos the agent may change. |
| Docs repo (optional) | `acme-co/handbook` | Company docs a pack can pin for the agent to read. |
| Operators team | `acme-co/agent-operators` | The people who may re-run a pack or ask for a revision. |
| Network hosts | `registry.npmjs.org` | What the agent's tools may reach, such as package registries. |
| Runner and model | `claude-code`, `claude-opus-5-5` | Which are available to you. Model use is billed to your own provider account (step 5). |

## Step 2: Make your packs repo

Create it from the template, and keep it private:

```sh
gh repo create acme-co/packs --private --template aypex-io/agent-pack-template
```

Or open [`aypex-io/agent-pack-template`](https://github.com/aypex-io/agent-pack-template)
and click **Use this template**. Its README is the full guide to writing packs. Your copy
also includes this guide, which you can delete once you're set up.

## Step 3: Install the App

1. Open <https://github.com/apps/aypex-agent-runs/installations/new>.
2. Choose your organization.
3. Choose **Only select repositories**, and pick your packs repo, your target repos and
   your docs repo if you have one.
4. Click **Install**.

The App can only see what you select. If you leave a repo out:

| Left out | What you'll see |
|---|---|
| The packs repo | No `pack-ready` check ever appears. |
| A target repo | The run fails after the merge, with a comment on the PR. |
| The docs repo | A pack that pins docs fails `pack-ready`, saying the App can't read the repo. |

The App asks for these permissions. For each run, the platform narrows them further:
it can read your packs and docs repos, and it writes code only to the one target repo
that pack names.

| Permission | Why |
|---|---|
| Contents: read and write | Read packs and docs; push the agent's branch to a target repo. |
| Pull requests: read and write | Open the agent's PR; read pack PRs. |
| Issues: read and write | Post status comments on PRs. |
| Checks: read and write | Post the `pack-ready` check. |
| Members: read | Confirm who is on your operators team. |
| Metadata: read | Required by GitHub for every App. |

## Step 4: Create the team and the labels

Create the operators team, and add the people who may re-run packs and ask for
revisions:

```sh
gh api -X POST orgs/acme-co/teams -f name=agent-operators -f privacy=closed
gh api -X PUT orgs/acme-co/teams/agent-operators/memberships/THEIR-LOGIN
```

Create the two labels. A label that doesn't exist can't be applied:

```sh
gh label create force:agent-run -R acme-co/packs
gh label create agent-revise -R acme-co/portal   # once for each target repo
```

## Step 5: Send Aypex your model API key

Runs use your own key, so model use goes straight on your bill with your provider. Claude
models are what's supported today, so this is an Anthropic API key.

1. **Make a key just for this.** In your Anthropic Console, create a workspace for Agent
   Runs, give it a spend limit, and create a key in it. A key of its own with a limit keeps
   any problem small, and you can revoke it without touching anything else.
2. **Encrypt it to Aypex's public key.** Install [age](https://age-encryption.org), then run:

   ```bash
   age -r age1qw7cps5np4j25aftjxhehxp6y2azqen8s7qrg0n7q0aaeataf3zqwk44fn -o model-key.age
   ```

   It waits for input and shows nothing. Paste the key, press Enter, then press Ctrl-D.
   This keeps the key out of your shell history.
3. **Send Aypex `model-key.age`,** by any route you like. Only Aypex can decrypt it. Delete
   your copy once Aypex confirms it arrived.

**To replace the key,** make a new one, send it the same way, and revoke the old one once
Aypex confirms. **To cut off access at any time,** revoke the key in your Anthropic Console.
Runs then stop with a message saying the key was rejected.

## Step 6: Wait for Aypex to register you

Tell Aypex that steps 2 to 5 are done. Aypex adds your organization to its registry and
tells you when your side is ready.

Until then, the App ignores you. Merges and labels start nothing and post nothing, and
that's expected.

**Check it's ready:** open a small PR on your packs repo that changes the README. Within
about 10 minutes it should have a `pack-ready` check reading **Ready: no pack changed**.
Close the PR afterwards.

## Step 7: Protect your packs repo

**Merging a pack starts a run,** so this ruleset decides who can start one. Without it,
a pack that fails its check can still be merged.

In your packs repo, go to **Settings**, **Rules**, **Rulesets**, and add a branch ruleset
on `main`:

- Set **Enforcement status** to **Active**, and leave the **Bypass list** empty.
- **Require a pull request before merging,** with at least one approval. Dismiss stale
  approvals when new commits are pushed.
- **Allowed merge methods:** merge commits only. A squash or rebase merge is refused
  after the fact, with a comment on the PR.
- **Require status checks to pass:** add `pack-ready`, with its source set to the
  `aypex-agent-runs` App (integration ID `5571914`). GitHub may not offer the App until
  the check has run, which is why this comes after step 6.

**Check it:**

```sh
gh api repos/acme-co/packs/rules/branches/main
```

You should see a `required_status_checks` rule with `integration_id` `5571914`, and a
`pull_request` rule with `allowed_merge_methods` of `["merge"]`.

## Step 8: Run your first pack

Start with something small, so you can see the whole path.

```sh
git switch -c pack/acme/first-pack
mkdir -p packs/acme
cp -R template packs/acme/first-pack
```

The branch must be `pack/<customer>/<slug>`, and the folder must match it. Fill in the
five files, removing every `FILL` marker. Your coding agent can write them with you, using
the `write-pack` skill in `.agents/skills/write-pack/` (in Claude Code, `/write-pack`). Start
the agent in your packs repo, or it won't find the skill. To update it later, replace that
folder with the newest one from `aypex-io/agent-pack-template`.

| File | What it says |
|---|---|
| `goal.md` | Why: the outcome, who it's for, how you'll know it worked. |
| `spec.md` | What: acceptance criteria, what happens when things go wrong, what's out of scope. |
| `plan.md` | How: the approach, the changes file by file, the tests. The agent follows it. |
| `AGENTS.md` | Standing rules for your code: stack, how to run tests, conventions. |
| `target.yaml` | Where: the repo, the model, permissions, the test command, the budget. |

The template's README has every `target.yaml` field. A small one looks like this:

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
budget: { max_turns: 60, deadline: 60m, usd_cap: 5 }
egress: [registry.npmjs.org]
```

If this is the first pack for this repo, ask Aypex to test its `verify` command in the
cluster before you merge. The test uses no model and writes nothing to GitHub.

Then:

1. **Push the branch and open a PR into `main`.** There's no local validator: the
   `pack-ready` check is the validator. Fix what it reports, and it runs again on each push.
2. **Review it like a design.** Merging approves the plan as written.
3. **Merge with a merge commit, as a person.** That starts the run.

## What you'll see

When the run finishes, a comment on the merged PR links to the agent's pull request, with a
cost table. A small pack takes a few minutes. The agent's PR is on your target repo, opened by the App. It holds a summary,
the agent's own review and the cost, and the platform re-runs your `verify` command itself
before opening it, so a failing test never becomes a PR.

Review it as you would any pull request. Then:

| You want to | Do this |
|---|---|
| Have the agent address your review comments | Leave the comments, then label the agent's PR `agent-revise`. |
| Run a merged pack again from scratch | Label the merged pack PR `force:agent-run`. |
| Change what was asked for | Write a new pack. A merged pack never changes. |

Only members of your operators team can use the labels. A label from anyone else is
refused.

## If something doesn't happen

| What you see | Why, and what to do |
|---|---|
| A merge posts nothing | The branch isn't `pack/<customer>/<slug>` in your packs repo, the PR didn't merge into `main`, or the person merging has a bot login or a login with an underscore. Otherwise, wait a few minutes, then ask Aypex. |
| A merge is refused, with a comment | The merge wasn't a merge commit, or the pack no longer passes. The comment says why. |
| `pack-ready` is missing | It appears within about 10 minutes. If it never does, check the App can see your packs repo, and that Aypex has registered you. |
| `pack-ready` says it couldn't run | The check failed, not your pack. Press **Re-run** on the `pack-ready` check itself. If it fails again, ask Aypex. |
| **Re-run all checks** does nothing | It isn't handled. Use **Re-run** on the `pack-ready` check itself. |
| A label does nothing | `force:agent-run` only works on a merged PR. A label already on a PR does nothing: remove it and add it again. |
| A PR from a fork fails | Packs must come from a branch in your packs repo. |
| A run stops: your API key was rejected | The key was revoked or isn't valid. Send a new one (step 5), then re-run the pack with `force:agent-run`. |
| A run stops: the model provider refused this request | Often the key's workspace can't use that model. Check the workspace's settings, then re-run the pack. |
| A run stops: out of credit | Your provider account or workspace has hit its balance or spend limit. Raise it, then re-run the pack. |
| A run stops: rate-limited, or the provider was unavailable | Nothing is wrong on your side. Re-run the pack later, or raise your limit with the provider. |

## What the agent can reach

- **Your repos:** only what you selected in step 3. The agent writes only to the target
  repo its pack names, on its own branch. It never writes to your packs repo, and it reads
  docs read-only.
- **Your code leaves GitHub:** for a run, your target repo and any pinned docs are copied
  to Aypex's cluster, and the agent sends what it reads to the model provider.
- **The network:** the agent connects only to the model's API and the hosts you agreed in
  step 1. DNS lookups aren't restricted, and package hosts can share addresses with other
  sites on the same CDN.
- **Your API key:** Aypex stores it in 1Password and in your namespace on the cluster. During
  a run a small proxy in its own sandbox holds it, and the agent has only a placeholder, so
  nothing the agent runs can read the key or send it anywhere. The agent can still spend
  through the proxy while a run is going, up to the pack's budget and your spend limit.
- **Isolation:** each run works in sandboxed containers, and each customer has its own
  namespace on the cluster.
- **Workflows:** the agent can't change `.github/workflows/`.
- **Budgets:** each pack sets its own limits on spending, turns and time.

## Removing it

Tell Aypex, so it can remove your registration and any stored secrets. Then uninstall the
App: in your organization's **Settings**, go to **GitHub Apps**, and uninstall
`aypex-agent-runs`. Your repos, packs and the agent's pull requests stay as they are.
