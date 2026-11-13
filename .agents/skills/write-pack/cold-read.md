# Cold read

You've been in the conversation, so you know things the pack doesn't say. The
run's agent won't. Before the PR, a reader who knows only what the run will know
reads the pack and says where it would have to guess.

Run it as a fresh agent on your strongest model, with nothing from this session
but the brief below. If your coding agent can't start one, give the person the
brief and ask them to paste it into a new session with the same agent, then
bring back the report.

Do at most two rounds. Fix what the first round finds, then run it once more.
Anything still open after the second round goes to the person.

## The brief

Replace `<PACK>` and `<REPO>` with the pack folder and the repo's checkout.

```text
You're about to carry out a piece of work alone, and nobody will be there to
answer questions. Before you start, read the brief you'll be given and tell me
where it falls short.

The brief is the folder <PACK>: goal.md, spec.md, plan.md, AGENTS.md,
target.yaml and refs/. The code is the repo at <REPO>. Read both, and nothing
else. Don't change anything, and don't run setup, installs or tests: reading is
enough.

How the work will run:
- You'll work alone in a copy of that one repo, on a new branch. You won't be
  able to open any other repo, so anything you need from elsewhere must be in
  refs/.
- You'll read goal.md, spec.md, plan.md and AGENTS.md, then follow plan.md.
- You may run only the shell commands that permissions.allow in target.yaml
  lists. Each bash: entry is a glob matched against the whole command, so
  bash:npm test allows npm test but not npm test with arguments, unless another
  entry ends in " *". A command with a variable in front never matches. The
  variables you need come from env.
- You can reach only the hosts listed under egress.
- If the brief is unclear in a way that changes what you'd build, you'll stop
  and ask instead of guessing, and the whole attempt is wasted.
- When you're done, a second reader who didn't see your work checks it against
  spec.md alone.
- Then the verify command in target.yaml runs in a clean Linux container, and
  the work is only accepted if it passes.

Report, under these headings, every place where:
1. Guesses: you'd have to guess what's wanted.
2. Two readings: the brief could be read two ways that lead to different code.
3. Untested: a criterion in spec.md has no test in plan.md, or its test would
   still pass with the code it covers removed.
4. Blocked: a command you'd need isn't allowed, needs a variable that isn't in
   env, or needs a host that isn't in egress.
5. Contradicts the code: a step in plan.md doesn't match the repo, such as a
   path that doesn't exist, a method that isn't there, or a pattern the repo
   doesn't use.
6. Missing from refs: you'd need something from another repo or document that
   isn't in refs/.

For each, give the file, a short quote, why it matters, and a suggested fix. If
a heading has nothing, write "None". If nothing at all needs fixing, write
"Nothing to guess".
```

## After the report

- A finding the code or the README can settle: fix the pack.
- A finding that needs a product decision: put it to the person, with your
  suggested answer, or under `## Open questions`.
- **Blocked** findings usually mean `target.yaml` needs a `bash:` entry, an
  `env` variable or an `egress` host. A host that isn't on the customer's agreed
  list can't be added by the pack: tell the person.
