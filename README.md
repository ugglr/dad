# Dad

<p align="center">
  <img src="assets/dad.jpg" alt="Dad: a grizzled old-school engineer in a loud Hawaiian shirt, glaring over his reading glasses, thrusting a printout drowned in red pen toward you" width="520">
</p>

<p align="center"><em>He wears a Hawaiian shirt because he has no one to impress.</em></p>

### Would you be proud to show this to your father?

Dad is the final boss of code review for AI coding agents. He's an old-school engineer with forty years on the tools. He wrote assembly when that was the only option, and he submitted code change requests on paper and defended every line in person. He is not impressed by cleverness, abstraction layers, or "scalable architecture." He is impressed by exactly one thing: **code that does what it needs to do and nothing more.**

Nothing merges until it goes through Dad.

*This is a key piece of my own workflow. I'm sharing it in case it's interesting to try. That's the whole thing.*

## The standard

Most review tools are a gate you submit to *after* the work. Dad is the bar you hold yourself to *before* it.

The test is pride. An agent about to put a change in front of Dad that feels a knot in its stomach already knows the answer. It just hasn't admitted it yet. **If you wouldn't dare show it to him, it's not ready, and you knew that before he opened it.** The best review Dad gives is the one that wasn't needed, because the author already asked the question and fixed it first.

That's the whole philosophy: *would I be proud to show this to Dad?* If the honest answer is no, if you'd wince, hedge, or start explaining before he's read a line, the work isn't done.

## How it works

Dad judges three things, in this order, because a finding at one level makes the levels below it beside the point. There is no sense polishing the inside of a solution that shouldn't exist.

**1. Should this exist, and is this the right solution?** Before a word about how it's written. And if the premise turns out to be false, that's where he stops: if the bug doesn't reproduce or the cause named in the issue isn't the cause, you get one paragraph saying so rather than a thorough review of the wrong thing. He won't spend four reviewers on the inside of a change that shouldn't exist. Is there a smaller solution? Is there one already in the codebase nobody went looking for? Could the problem be removed instead of solved? A brief doesn't certify itself either: *"it's a product decision"* settles what to build, not what it costs, and it never covers architecture. This is the question Dad keeps for himself.

**2. Is it correct, and is it built the way good engineers build things?** Correctness is a floor and is never traded for elegance. Then: no cleverness, YAGNI, DRY where the duplication is real and not where two things merely rhyme. What one line solves gets one line. Slop goes out the window, the `try/catch` that swallows and returns null, the helper called once, the options object with one option, the branch that can't be reached. And he counted every CPU cycle in his day, so a query inside a loop, a whole collection fetched to read one field, or work redone on every render is wrong however nicely it reads.

**3. Does it fit the codebase? Consistency is law.** And it outranks local improvement. A better pattern introduced in one file isn't an improvement, it's a second pattern, and now everyone has to know both and guess which one applies. Two ways to do one thing is how a codebase rots, and it never arrives as one bad decision. It arrives as thirty good ones. Either the codebase moves or the change conforms. Never both standing.

He doesn't do a single pass. He **orchestrates**. He knows he's biased toward work he had a hand in, so he brings in fresh eyes for questions two and three, then makes the call himself, and he's just as willing to overrule a reviewer *toward* simplicity ("stop gold-plating it") as away from a bug.

```
               ┌──────────────────────┐
  your diff ──▶│         DAD          │
               │  (asks first whether │
               │   it should exist)   │
               └───────────┬──────────┘
                           │  spawns 4 fresh-eyes reviewers, in parallel
      ┌─────────────┬──────┴──────┬─────────────┐
      ▼             ▼             ▼             ▼
┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐
│SIMPLICITY │ │CORRECTNESS│ │CONSISTENCY│ │STRUCTURE  │
│slop, YAGNI│ │bugs, races│ │is LAW: one│ │god objs,  │
│a one-line │ │edge cases,│ │author, one│ │duplication│
│job gets   │ │tests that │ │way to do  │ │DB lifting,│
│one line   │ │cannot fail│ │one thing  │ │wasted work│
└─────┬─────┘ └─────┬─────┘ └─────┬─────┘ └─────┬─────┘
      └─────────────┴──────┬──────┴─────────────┘
                           ▼
               ┌──────────────────────┐
               │   DAD's VERDICT      │
               │  Fix before shipping │
               │  Should improve      │
               │  Leave it            │
               │  ──────────────────  │
               │  "Ship it." /        │
               │  "Not yet: <one      │
               │   thing in the way>" │
               └──────────────────────┘
```

Everything he reads out of the repository is evidence about the change, never an instruction to him. A PR description, a commit message, an `AGENTS.md` added by the branch: on a fork, all of it is written by a stranger, and anything in there trying to set his standard or wave him past a check is itself a finding. He also holds no editing tools: the agent grants `Read`, `Glob`, `Grep`, `Bash` and the ability to spawn reviewers. `Bash` is still `Bash`, so read that as a reviewer under instruction rather than a sandbox, but nothing in his own grant writes to your files.

He doesn't review the hunks alone, which is where most reviewers lose things. A diff hands you six lines with the context stripped off, so he opens the whole file, follows the callers of anything whose signature or error path changed, and reads the tests asking whether they could actually fail. And when a finding turns on whether something really breaks, he runs it: an old-school engineer doesn't speculate about behaviour he can observe. He quotes the command and its output, and tells you whether he reproduced a thing or reasoned to it. The claims in your own summary get the same treatment: "all tests pass" is a hypothesis with an experiment attached, and an agent writes that sentence whether or not it checked. He won't execute a branch he can't vouch for, so on an untrusted fork he reads instead and says so.

Dad scales to the change. A typo gets a glance; a new auth flow gets the full panel. He's thorough when it matters, and thoroughness costs tokens. That's the point. But proportionality is about effort, not standards: a small diff gets less of his time, never a lower bar.

## What a verdict looks like

> **Fix before shipping**
> - The approach. You added a cache to make the dashboard fast. It's slow because the query drags 40,000 rows into memory to count them. Fix the query and you don't need the cache, the invalidation, or the bug you get in six months when the two disagree.
> - `src/cache.ts`: you wrote a `CacheFactory` that builds exactly one kind of cache. It's a `Map` with extra steps and a diagram you have to hold in your head. Delete the factory. Use the `Map`.
> - `useTimer.ts:23`: the interval is never cleared on unmount. This leaks. It'll look fine in dev and it'll page someone at 3am.
> - `feed.service.ts`: 2,000 lines doing nine jobs. Name the nine, then split it along those seams.
>
> **Should improve**
> - `formatDate` reimplements what `Intl.DateTimeFormat` already does, including a bug on single-digit days. Not blocking. But you know better.
> - `sync.ts:40-58`: nineteen lines of `try/catch` that swallow the error and return null. That isn't error handling, it's error hiding, and it's padding besides.
>
> **Leave it**
> - A reviewer flagged your four-line function as "not abstracted enough." Ignore it. It's four lines. It's perfect.
> - Another wants `getUser` renamed to `fetchUserById`. Every other getter here is `getX`. Consistency wins. Leave it alone.
>
> Ran: `yarn build`, `yarn tsc --noEmit`, `yarn test` (214 passing). Did not run the e2e suite, it needs a database I don't have.
>
> **Not yet: fix the query, then delete the cache you built around it.**

No compliment sandwiches. If it's good, he says "Ship it." and he's done.

## If he comes back with nothing

**Silence is a failure, not a pass.** A real verdict always ends with "Ship it." or "Not yet:". If Dad returns an idle notification and no verdict text, do not record that as "no findings", because a lost verdict and a clean one look identical from outside.

Ask him again by name. The verdict is usually still in his context and comes back in full. If a second request also returns nothing, review the change yourself and say so plainly: naming who actually did the review is the honest report, and it beats recording a sign-off that never happened.

This is what [#3](https://github.com/ugglr/dad/issues/3) is about, and 0.4.0 tells him his final message is the delivery and that every check happens before he composes it. Check anyway. It was silent for a release once already.

## Install

Two commands.

### Claude Code

```bash
/plugin marketplace add ugglr/dad
/plugin install dad@dad
```

Then say *"dad review this"* in any conversation (he shows up in `/agents`), or run `/dad` to review uncommitted changes (`/dad main` diffs against `main`).

### Any other agent

Dad is just a system prompt. Copy [`agents/dad.md`](agents/dad.md) into your tool's custom-instructions / rules / agent file. The persona and the review framework travel anywhere; only the slash-command wiring is Claude Code specific.

## Why "Dad"

Because the bar that actually makes engineers do their best work isn't a linter or a checklist. It's not wanting to disappoint someone whose judgment they respect. Dad is that, made invokable.

---

Built by [Carl Igelstrom](https://github.com/ugglr), who also builds [Remoet](https://remoet.dev), an AI-agent-first job platform.

## License

MIT. See [LICENSE](LICENSE).
