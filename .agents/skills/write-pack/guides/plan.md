# Writing `plan.md`

The plan is how the run builds the spec. The run's agent follows it, and
merging the pack approves it, so people review it as they would a design.

Write a plan only when the code it changes is there. A pack that waits for
another pack keeps the template's `plan.md` until the earlier pack's agent PR
has merged into its repo (see [split.md](../split.md)).

Read the code before you write the plan. Fill the template's sections and delete
every `FILL` comment.

## Approach

A few sentences: what will be built, and why this way rather than another. Name
the existing file it follows, by path. The run's agent copies what you point it
at.

## Changes

File by file: the path, and what changes in it. Check every path exists in the
repo, or mark it **new**. Say where a change follows an existing pattern, and
which.

## Tests

One test, at least, for every acceptance criterion. For each one give:

- its name and the file it goes in;
- what it asserts;
- **why it fails without the code**: which line or branch, removed or broken,
  makes it fail.

A test that still passes with the code removed proves nothing. Watch for:

- assertions that hold either way, like checking a list is empty when it starts
  empty;
- test setup that already does the very thing the code is meant to do, so the
  test can't tell whether the code did it;
- mocks that replace the code under test.

**For tests that guard data or access**, such as permissions or who can see
whose data, add a step to the plan: break the code on purpose (remove the
scoping line), run the test, see it fail, then restore the code. Say which line
to remove.

If the spec's "Existing behaviour that must not change" lists cases the existing
tests don't prove, the plan adds a test for each.

## Risks and open questions

What could go wrong, and anything the plan isn't sure of. If something here
would change what gets built, it's a spec question: move it to the spec's
`## Open questions` and ask. Write "None" if nothing applies, and say why.

## Out of scope

What the plan leaves alone, matching the spec's.
