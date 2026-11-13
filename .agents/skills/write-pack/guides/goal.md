# Writing `goal.md`

The goal is why the work matters. Only people know it, so this is the one file
you mostly ask about rather than find in the code.

Draft it from the request, then ask the person to correct it. Suggest; don't
interrogate. Fill the template's three sections:

- **The goal:** one paragraph on the outcome, in the customer's words. Why it
  matters and to whom. Not how to build it.
- **Who it's for:** the people who will use or notice the change.
- **How we'll know it worked:** what can be observed after release. A task a
  user can now do, a support ticket that stops arriving, a number that moves.

The run's agent uses the goal to make the small calls the spec doesn't cover,
the way the people asking would. A vague goal gets generic calls.

## When the work is split

Packs from one request share the why. Write it once in the same words in each
pack, then add a sentence saying what this pack's part is, and which pack
completes the outcome. For example: "This pack adds the export endpoint. The
customer sees the button when `acme/export-csv` lands."
