# Product Agents

Subagents for [Claude Code](https://claude.com/claude-code) that review
**product decisions**, not code. They run before anything is built, while
changing the shape of a feature is still free.

## Agents

| Agent | What it asks | When to run it |
| --- | --- | --- |
| [`jtbd-critic`](agents/jtbd-critic.md) | Who now has to do something they didn't before, how often, and did anyone agree to that? | When a feature is still a sentence, ticket or plan |

### jtbd-critic

Code is paid for once; work you hand to users is paid every time it fires, by
everyone it fires for. The JTBD critic lists every job-to-be-done a proposed
feature creates (role, trigger, frequency, cost) and applies one rule:

> A net-new job ships only if it (a) requires judgment the system provably
> cannot make, or (b) retires a job of equal or greater weight.

Anything else is refused, together with the reshape that makes the job
disappear: derive the value, default it, infer it, ask once at setup, or drop
the surface.

It returns a verdict, a jobs table, reshapes, a short list of questions only
the requester can answer, a plain-language **jobs sheet**, and a **mockup
brief** that names every field on screen and where its data comes from.

## Install

Copy an agent into your project's (or your user-level) agents directory:

```bash
mkdir -p .claude/agents
curl -o .claude/agents/jtbd-critic.md \
  https://raw.githubusercontent.com/jlindqui/product-agents/main/agents/jtbd-critic.md
```

Then ask Claude Code to use it:

```
Run the jtbd-critic on this: "When an invoice is disputed, ask the
account owner to pick a reason code before it can be paid."
```

## Make it yours

The agents are written to work on any product, but they are sharper with
context. The JTBD critic reads, if present:

- a product context file (`PRODUCT.md`, `CLAUDE.md`, or similar) describing who
  the product serves, and
- wherever your code defines user roles.

Replace the generic roles section with your product's real roles, and add your
own past decisions as examples. A critic that knows "we chose more asks here,
and it was right" rules better than one that doesn't.

## License

[MIT](LICENSE)
