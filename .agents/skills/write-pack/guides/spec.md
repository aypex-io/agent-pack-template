# Writing `spec.md`

The spec is what the run builds and what its review checks the work against.
Where it is silent, both are blind. Spend most of your effort here.

Write the summary and the acceptance criteria first, then do the failure pass,
then the rest. Delete every `FILL` comment as you go.

## Acceptance criteria

- **Each criterion can be checked**: by a test, or by a reviewer looking at the
  result. "Works well" can't be checked. "Returns 404 for an unknown id" can.
- **One behaviour per criterion.** Two behaviours joined by "and" are two
  criteria.
- **Name the place in the code** a criterion is about, when there is one: the
  endpoint, the job, the setting. The run's agent shouldn't have to guess which
  of two similar things you meant.
- **Use the repo's words.** Name records, fields and settings as the code names
  them.
- Keep the template's format: `- [ ] ` and the criterion. `pack-ready` needs at
  least one real criterion under `## Acceptance criteria`.

## The failure pass

This is the step that decides whether the run's PR merges without a revision.
Defects that reviewers find after a run are rarely on the expected path. They
are in what happens around it, which a passing test run can't show.

Go through every kind below. For each one, **look in the code** for where it
applies to this change, using the hints. Then:

- If it applies, it is a **decision**. Put it to the person with the answer
  you'd suggest, and why. Their answer becomes a criterion, and an example if
  that helps.
- If nobody can answer it yet, write it under `## Open questions`, with your
  suggested answer. `pack-ready` blocks the merge until it's answered.
- Either way, write a line for it under "When things go wrong": the decision
  and the criterion that holds it, or "None, because …" with a reason when the
  kind doesn't apply.

Never write your own answer in as if it were decided. The run can't tell an
invented answer from a decided one, and it builds whatever the spec says.

**Repeats.** The same thing happens twice: a job retried, a webhook redelivered,
a form submitted twice, a message delivered at least once. Look for: jobs and
their retry settings, webhook handlers, anything triggered by an event. Ask:
what happens the second time, and what must not happen twice?

**Overlap.** Two things change the same record at once: two jobs for the same
record, a user editing while a job writes, two requests racing. Look for: jobs
keyed by a record, writes without a lock or a unique constraint, "find or
create". Ask: which one wins, and what must never be lost or duplicated?

**Undo mid-flight.** Someone cancels, revokes, deletes or unpublishes while work
on that thing is queued or running. Look for: every way the thing can be
cancelled, revoked, deleted, hidden or switched off, and every job or request
that works on it. Ask: what must the work in progress do when it finishes, and
what must no longer exist afterwards?

**Partial failure.** A call fails halfway: an external service errors, times
out or returns rubbish, or a step after the first write fails. Look for:
external calls, multi-step writes, anything retried. Ask: what state is left,
what does the record show, what is retried, and what must stop retrying because
it can never succeed (a bad setting, a missing record) and leave a trace saying
why?

**Odd input.** Values that parse but mean something else, and states that are
legal but unusual: different casing, formats and separators for the same value,
blanks and whitespace, a value that parses but means something unexpected,
empty lists, very long values, and records in every state they can be in
(archived, deleted, half-finished, created before a rule existed). Look for: where the change reads input or
settings, and every state the records it touches can be in. Ask: what does each
one do?

**Production defaults.** Anything that behaves differently in production
because a value isn't set: a fake or test implementation as the default, a
missing setting that silently picks something, a development shortcut left on.
Look for: defaults in config and environment files, startup code, and
fallbacks such as `value || default`. Ask: what happens in production when nothing is configured?
The answer should never be "the test double runs".

**Existing behaviour that must not change.** A change that claims "everything
else behaves as before" is only as good as the cases it names. Existing tests
may not cover them. Look for: every option, parameter, flag, mode and caller of
the code you're changing. List each one in a criterion that says it must keep
working, and require a test for each one the existing tests don't already prove.
Check the existing tests, rather than assuming they cover it.

**Access and scoping.** Who may see or change what: permissions, roles, and
data that belongs to one account, organisation or user. Look for: account or
organisation scoping, policies, row-level rules, and background work, such as
jobs and scheduled tasks, which runs outside a request. Ask: what must each kind of user never see or
change? Every criterion here needs a test that fails when the scoping code is
removed. Say so in the criterion.

**The seam between packs** (only when the work is split). The packs land one
after another, so for a while one side is new and the other old. Ask: what
happens when the new side meets the old one, in each direction?

## Interface

Only when the work is split into several packs: the interface agreed with the
other packs, word for word as in every other pack's spec (see
[split.md](../split.md)). Don't paraphrase it.

## Examples

Concrete inputs and expected outputs, sample requests and responses, or a short
test sketch. An example settles what a sentence leaves open. Put large ones in
`refs/`.

## Out of scope

What the run must not change, even if it looks related: nearby bugs, refactors,
dependency upgrades, the follow-up packs. The run's agent is told to stay inside
the spec, so name the things it might be tempted by.

## Constraints

Compatibility, versions to support, libraries to use or avoid, migrations that
must stay backward compatible, performance limits. Delete the section if there
are none.

## Open questions

Everything nobody could decide yet, each with your suggested answer, and the
"Waits for …" line of a pack that depends on another. Leave the section out
when it's empty.
