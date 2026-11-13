# Spec

## Summary

<!-- FILL: two or three sentences. What changes, in plain terms. -->

## Acceptance criteria

Each criterion must be checkable by a test or by a reviewer looking at the
result. The agent's plan maps a check to every one. A test only counts if it
fails when the code it covers is removed or broken.

<!-- FILL: replace these with real criteria, then delete this comment. -->
- [ ] Given …, when …, then …
- [ ] …

## When things go wrong

<!-- FILL: what must happen off the expected path. Most defects a
     code review finds are here, and a passing test run cannot show them.
     Answer each question that applies, and add a criterion above for each answer:
     - Repeats: a job or request runs twice (retries, redelivery, double submit).
     - Overlap: two requests or jobs change the same record at once.
     - Undo mid-flight: a user cancels, revokes or deletes while work is in progress.
     - Partial failure: a call fails halfway. What state is left, and what retries?
     - Odd input: casing, blanks, separators, values that parse but mean something else.
     - Production: defaults, fakes or test engines that must never run there.
     - Unchanged behaviour: what must keep working as it does today. Name each
       case, because the existing tests may not cover it.
     - Access: who may see or change what, such as tenant or permission scoping.
     Write "None" if nothing applies, and say why. -->

## Examples

<!-- FILL: concrete inputs and expected outputs, sample requests and responses,
     or screenshots in refs/. Examples beat descriptions. -->

## Out of scope

<!-- FILL: what the agent must not change, even if it looks related. -->

## Constraints

<!-- FILL: performance limits, compatibility, libraries to use or avoid,
     migrations that must stay backward compatible. Delete if none. -->

## Open questions

<!-- Anything unresolved. A pack with open questions here should not be merged
     until they're answered, because merging starts the run. Delete this section when empty. -->
