---
name: write-pack
description: Use when writing a pack for the Agent Run Platform, or finishing one that isn't merged yet (an open question answered, a reviewer's edits, a pack-ready failure, or an earlier pack it waited for has merged). Gets what a coding agent needs into goal.md, spec.md, plan.md, AGENTS.md and target.yaml, so the run's pull request can merge without a revision.
metadata:
  version: "2026-11-13"
  source: https://github.com/aypex-io/agent-pack-template/tree/main/.agents/skills/write-pack
---

# Write a pack

A pack is the brief a coding agent works from when the Agent Run Platform runs
it against one repo. You write it with the person who wants the work done.

## What success is

The pull request the run opens merges as it is, without a revision. Passing
`pack-ready` is the floor, not the goal.

Runs rarely go wrong on the expected path. They go wrong where the spec said
nothing: two things happening at once, a failure halfway through, an odd input,
a default that reaches production, behaviour that should have stayed the same,
a test that still passes with the code removed. The run's own review checks the
work against `spec.md`, so where the spec is silent, the review is blind too. An
ambiguous spec costs more: the run's agent stops and asks, and the whole run is
spent. Most of your effort goes into the spec.

## Rules

- **Never decide a product question quietly.** Put each one to the person, with
  the answer you'd suggest. If nobody can answer it yet, write it under
  `## Open questions` in `spec.md`. With the merge rules the README describes,
  `pack-ready` blocks the merge while that section has text. An invented answer
  reads exactly like a decided one, and the run builds it.
- **Ask people only what people know:** why the work matters and what should
  happen. Find everything else in the code.
- **Point at the repo's own guides.** Don't copy them into the pack. Copies go
  stale.
- **Never merge a pack PR, apply `force:agent-run` or `agent-revise`, or start a
  run.** Merging is the approval, and a person does it.
- **Never edit a merged pack.** If it was wrong, write a new pack.
- **Read only this packs repo and the repos on the map** (see
  [split.md](split.md)). Facts about the platform, such as the allowed `egress`
  hosts, come from the pack rules, earlier packs or the person, not from other
  folders on this machine.
- **Keep secrets out.** Don't open secret files (`.env*`, credentials, keys).
  Never put a secret value anywhere in a pack: a pack stays in git history for
  good, and a merged pack can't be edited. If the tests need a secret, tell the
  person: the pack can't carry it.
- **Don't change the target repos.** Read them, and run their own setup and test
  commands. Read a setup script before you run it, and don't run one that
  installs system software, starts services or resets a database: ask instead.
  If running commands changes a tracked file or leaves new untracked files,
  tell the person, and restore or remove them only with their OK. If a checkout
  is on another branch, behind its `base` or has local changes, tell the person
  and ask: don't switch, pull or reset it yourself.

## The pack rules

The rules a pack must meet are in the template's README. Read the current
version from
`https://raw.githubusercontent.com/aypex-io/agent-pack-template/main/README.md`.
If you can't reach it, read `README.md` in this repo, if it is still the
template's guide. If you have neither, tell the person, carry on, and rely on
`pack-ready` to catch what you miss.

## How the work is shared out

```
write-pack  (this session: talks to the person; your strongest model)
├─ guides/  goal · spec · plan · agents-md · target-yaml   ← load each as you write that file
├─ repo survey ─► a separate agent per changed repo, your cheapest capable model
│                 drafts AGENTS.md + target.yaml while you discuss the goal
├─ goal → spec (failure pass, the person decides) → plan   ← in order, in this session
└─ cold read ───► a fresh agent, your strongest model, sees only the pack and its repo
```

The separate agents are optional. If you can't start one, do the survey
yourself, in the same order. The cold read is the exception: you know too much
to read the pack cold, so [cold-read.md](cold-read.md) says how to hand it to
the person.

## Steps

1. **Start.** Take whatever the person has: a ticket, an issue, a paragraph.
   Find the customer slug from the folders under `packs/`, or ask for it. If the
   pack already exists and isn't merged, see
   [Picking up a pack in progress](#picking-up-a-pack-in-progress). A folder
   that still holds the template's `FILL` markers is a new pack: start here.
2. **Map the repos** the work touches, and say which change and which are only
   read. Load [split.md](split.md).
3. **Split the work** into packs: one per changed repo, in order, with the
   interface between them agreed once. Also in [split.md](split.md).
4. **Survey each changed repo** and draft its `AGENTS.md` and `target.yaml`.
   Load [survey.md](survey.md). It uses
   [guides/agents-md.md](guides/agents-md.md) and
   [guides/target-yaml.md](guides/target-yaml.md). Run it as a separate agent
   per repo if you can, while you work on the goal. A separate agent can't ask
   the person anything, so it lists its questions and warnings under
   **For the person** in its report. Raise them before the spec is done.
5. **Goal.** Load [guides/goal.md](guides/goal.md).
6. **Spec**, for each pack, including the failure pass. Load
   [guides/spec.md](guides/spec.md).
7. **Plan**, for each pack whose earlier packs' agent PRs have merged into
   their repos. Load
   [guides/plan.md](guides/plan.md).
8. **Cold read** of each pack that has a plan. Load
   [cold-read.md](cold-read.md).
9. **Check the pack rules and open the PRs**, if the person asks. Load
   [checks.md](checks.md).

## Picking up a pack in progress

A pack folder that exists and isn't merged comes back for a reason: an open
question was answered, a reviewer pushed edits, `pack-ready` failed after the
last session, or the agent PR of a pack it waited for has merged. First fetch
the pack's branch and bring your copy up to date, so you see what reviewers
pushed. Never force-push over their work. Then find out which it is, read what
changed (`git log` and `git diff` on the branch), and redo only what the change
affects:

- a new or changed criterion gets the failure pass in
  [guides/spec.md](guides/spec.md) and a test in the plan;
- a changed `target.yaml` or `AGENTS.md` gets checked against the repo again,
  as in [survey.md](survey.md);
- a pack that was waiting: check that the earlier pack's agent PR has merged
  into its repo's `base`, and that any release it needs is out. Check that the
  code as merged still matches the agreed `## Interface`. If it doesn't, update
  the interface in every pack that carries it, and cold-read them again. Then
  write the plan against the code as it is now, remove the "Waits for …" line
  from `## Open questions`, and tell the person to mark the draft PR ready for
  review;
- a `pack-ready` failure gets the fix it names, except text under
  `## Open questions`: answering those is the person's job, never yours;
- anything that changed the spec or plan gets a new cold read.

Leave everything else as it is. A cold read looks at the whole pack, so it can
find gaps outside what changed. Fix those too, but list them for the person:
anyone who already approved the pack needs to look again. If the pack is
already merged, don't edit it: write a new pack.

## How to ask

Ask few questions, and suggest an answer for each, so the person can just say
yes. Gather the questions for a step and ask them together, rather than one per
message. Never ask what the code can tell you. If your agent has a way to put
choices in front of the person, use it. If not, a numbered list works.

## Updating this skill

`metadata.version` above is this copy's date. The source is
`.agents/skills/write-pack/` in `aypex-io/agent-pack-template`. To update,
replace this folder with that one, so files removed there go too. If the skill
was copied into `.claude/skills/write-pack` instead of linked, replace that copy
as well.
