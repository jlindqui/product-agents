# Product Agents

Claude Code subagents for the step between **spec** and **build**.

Most review happens after the code is written. By then the shape of the
feature is settled: the screens exist, the fields exist, and the work they ask
of people is already decided. Code review checks that the code is correct. It
does not check that the feature will achieve what it was for.

Product agents review the spec instead. They run while the feature is still a
sentence, a ticket or a plan, and ask whether what is about to be built
actually achieves the goal, and at what cost to the people who will use it.
Changing the shape at that point costs a sentence, not a rewrite.

```
idea → spec → [ product agents ] → build → code review → ship
```

## What a review checks

- **The goal.** What the feature is for, in the requester's own words. Every
  finding is measured against it.
- **The work it creates.** Every new task the feature hands to a person: who,
  triggered by what, how often. A feature that meets its goal by giving a
  hundred users a new daily chore has not met it.
- **The shape.** Where the system could do the work itself (derive the value,
  default it, ask once instead of every time), the review says so before it is
  built the expensive way.
- **What to build.** A plain-language list of the jobs the feature serves, and
  a mockup brief naming every field on screen and where its data comes from,
  so the design can be checked before anything is wired.

## Agents

| Agent | Question it answers |
| --- | --- |
| [`jtbd-critic`](agents/jtbd-critic.md) | Who now has to do something they didn't before, how often, and did anyone agree to that? |

More will follow, each covering one question that belongs between spec and
build.

### jtbd-critic

Code is paid for once; work handed to users is paid every time it happens, by
everyone it happens to. The JTBD critic lists every job-to-be-done a proposed
feature creates and applies one rule:

> A net-new job ships only if it (a) requires judgment the system provably
> cannot make, or (b) retires a job of equal or greater weight.

Anything else is refused, with the reshape that makes the job disappear.

It is not a scope-cutter. When a person genuinely has to decide something, the
critic keeps the job; the point is that the workload becomes a decision rather
than an accident.

Each review returns:

1. **A verdict:** ship as proposed, ship with reshapes, or reshape first.
2. **A jobs table:** role, trigger, frequency, cost and verdict for each job.
3. **Reshapes:** the change that removes each refused job.
4. **Questions only the requester can answer**, if any.
5. **A jobs sheet** for the person who asked, in plain words.
6. **A mockup brief:** screens, states, every field and its source, and sample
   data to review a design against.

It also works after the fact. Point it at a merged pull request and it runs a
retrospective audit, with the reshapes as follow-up work.

## Install

Copy an agent into your project's `.claude/agents/` directory, or into
`~/.claude/agents/` to use it everywhere:

```bash
mkdir -p .claude/agents
curl -o .claude/agents/jtbd-critic.md \
  https://raw.githubusercontent.com/jlindqui/product-agents/main/agents/jtbd-critic.md
```

## Use

Hand it the spec, in whatever form it exists:

```
Run the jtbd-critic on this: "When an invoice is disputed, ask the account
owner to pick a reason code before it can be paid."
```

```
Run the jtbd-critic on the plan in docs/plans/bulk-import.md
```

```
Run the jtbd-critic on PR #412
```

## Make it yours

The agents work on any product, but they are sharper with context. The JTBD
critic reads, if present:

- a product context file (`PRODUCT.md`, `CLAUDE.md`, or similar) describing
  who the product serves and what it is for, and
- wherever your code defines user roles.

Replace the generic roles section with your product's real roles, and add your
own past decisions as examples. A critic that knows "we chose to ask more here,
and it was right" rules better than one that doesn't.

## License

[MIT](LICENSE)
