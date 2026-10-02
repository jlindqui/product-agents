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
spec → jtbd-critic → you agree the jobs → wireframer → you agree the UI → build
```

## How it works

1. **Review the jobs.** Hand the spec to the `jtbd-critic`. It writes
   `reviews/<feature>/jtbd.md`: what the feature is for, every job it creates
   for people, which jobs it refuses and how to reshape them, and a mockup
   brief for anything people will see.
2. **Agree the jobs.** Read the review. Accept the reshapes, answer its
   questions, or send it back. Nothing is drawn until the jobs are agreed, so
   the screens come from the reshaped feature, not the first draft.
3. **Review the UI.** Hand the agreed review to the `wireframer`. It writes
   `reviews/<feature>/wireframe.html`: a clickable wireframe, in your
   product's look when it can match it, with
   every screen and state reachable, each field tagged with where its data
   comes from, each screen tied to the jobs it serves, and a panel listing
   gaps and unserved jobs.
4. **Agree the UI, then build.** Both files stay in the repository as the
   record of what was decided and why.

Changes with nothing a person sees stop after step 2.

## What a review checks

- **The goal.** What the feature is for, in the requester's own words. Every
  finding is measured against it.
- **The work it creates.** Every new task the feature hands to a person: who,
  triggered by what, how often. A feature that meets its goal by giving a
  hundred users a new daily chore has not met it.
- **The shape.** Where the system could do the work itself (derive the value,
  default it, ask once instead of every time), the review says so before it is
  built the expensive way.
- **The screens.** Whether every agreed job has a place on screen where it
  gets done, and whether every field on screen has a real data source.

## Agents

| Step | Agent | Question it answers | Writes |
| --- | --- | --- | --- |
| 1 | [`jtbd-critic`](agents/jtbd-critic.md) | Who now has to do something they didn't before, how often, and did anyone agree to that? | `reviews/<feature>/jtbd.md` |
| 2 | [`wireframer`](agents/wireframer.md) | Does every agreed job have a place on screen, and does every field have a real source? | `reviews/<feature>/wireframe.html` |

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

Each review contains:

1. **A verdict:** ship as proposed, ship with reshapes, or reshape first.
2. **A jobs table:** role, trigger, frequency, cost and verdict for each job.
3. **Reshapes:** the change that removes each refused job.
4. **Questions only the requester can answer**, if any.
5. **A jobs sheet** for the person who asked, in plain words.
6. **A mockup brief:** screens, states, every field and its source, and sample
   data. This is the wireframer's input.

It also works after the fact. Point it at a merged pull request and it runs a
retrospective audit, with the reshapes as follow-up work.

### wireframer

Draws the agreed mockup brief as one self-contained HTML file. When it runs
inside a project with a UI, it matches the product: its colours, type,
components and page layout, copied from the project's own styles and existing
screens, so the reviewer sees what users will see. With no UI to match, it
draws in greyscale rather than inventing a brand. It uses the brief's own
sample data, makes filters and expandable rows actually work, and overlays
review notes (switchable off to see the screen clean):

- the source of every field, with fields the brief marks NEW highlighted and
  anything it had to invent tagged **NOT IN BRIEF**;
- the jobs each screen serves;
- why a reshape removed something, so nobody asks for it back.

A review panel lists every job with the screens that serve it, flags any job
no screen serves, and collects the gaps and assumptions. It will not draw a
feature whose jobs review was refused and not yet reshaped.

## Install

Copy the agents into your project's `.claude/agents/` directory, or into
`~/.claude/agents/` to use them everywhere.

macOS / Linux / Git Bash:

```bash
mkdir -p .claude/agents
for a in jtbd-critic wireframer; do
  curl -o .claude/agents/$a.md \
    https://raw.githubusercontent.com/jlindqui/product-agents/main/agents/$a.md
done
```

Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force .claude/agents | Out-Null
'jtbd-critic','wireframer' | ForEach-Object {
  Invoke-WebRequest -OutFile ".claude/agents/$_.md" `
    "https://raw.githubusercontent.com/jlindqui/product-agents/main/agents/$_.md"
}
```

**Then wire it in** — see [Wire it into your project](#wire-it-into-your-project).
Copying the files alone will not make the review happen; it only makes it
possible to ask for by name.

## Use

Hand the critic the spec, in whatever form it exists:

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

Once you have agreed the jobs:

```
Run the wireframer on reviews/invoice-dispute-reasons/jtbd.md
```

## Make it yours

The agents work on any product, but they are sharper with context. Both read,
if present:

- a product context file (`PRODUCT.md`, `CLAUDE.md`, or similar) describing
  who the product serves and what it is for, and
- wherever your code defines user roles.

The wireframer also reads your design tokens, theme and existing screens in
the same area, so the wireframe looks like your product and uses the
components your users already know.

Replace the critic's generic roles section with your product's real roles, and
add your own past decisions as examples. A critic that knows "we chose to ask
more here, and it was right" rules better than one that doesn't.

### Wire it into your project

⚠️ **Do this part.** Installing the files makes the agents *available*; it does
not make them *happen*. Nothing tells Claude the critic exists, so until you
add the block below you have to remember to type "run the jtbd-critic on…"
every single time — and the one time it matters most is the time you are in a
hurry and forget. The second half matters for a different reason: a subagent
reports to the main Claude session, not to you, and that session can summarise
away the critic's `NEXT` line.

Add this to your project's `CLAUDE.md` (or `AGENTS.md`, or whatever file your
assistant reads on every session):

```markdown
## Product review

- **Before building anything a person will use** — a new screen, field, form,
  button, queue or notification, or any change to what someone has to do —
  run the `jtbd-critic` on my request FIRST, before writing code or a plan.
  Treat whatever I said as the spec; do not ask me to write a longer one.
  Skip it only when the change creates no new decision, reading or data entry
  for anyone.
- After a jtbd-critic review, always show me its NEXT step and the path to the
  review file. When the review has a mockup brief, offer to run the wireframer
  once I have agreed the jobs. Do not run it before then.
- After a wireframer run, show me the path to the wireframe and its JOBS NOT
  SERVED and GAPS lines.
```

The agent is called by the `name:` in its own front matter, which matches the
filename — so if you rename `jtbd-critic.md`, change the name inside it and in
this block too, or the routing silently stops working.

## License

[MIT](LICENSE)
