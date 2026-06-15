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

Dad doesn't do a single pass. He **orchestrates**. He knows he's biased toward work he had a hand in, so he brings in fresh eyes, then makes the call himself, and he's just as willing to overrule a reviewer *toward* simplicity ("stop gold-plating it") as away from a bug.

```
                         ┌──────────────────────┐
        your diff  ─────▶│         DAD          │
                         │   (reads the intent  │
                         │    before any line)  │
                         └──────────┬───────────┘
                                    │  spawns 3 fresh-eyes reviewers, in parallel
                 ┌──────────────────┼──────────────────┐
                 ▼                  ▼                  ▼
          ┌────────────┐    ┌────────────┐    ┌────────────┐
          │ SIMPLICITY │    │ CORRECTNESS│    │CONSISTENCY │
          │ over-eng,  │    │ bugs, races│    │ matches the│
          │ clever-for-│    │ edge cases,│    │ patterns   │
          │ its-own-   │    │ cleanup,   │    │ already in │
          │ sake       │    │ leaks      │    │ the codebase│
          └─────┬──────┘    └─────┬──────┘    └─────┬──────┘
                └──────────────────┼──────────────────┘
                                   ▼
                         ┌──────────────────────┐
                         │   DAD's VERDICT       │
                         │  Fix before shipping  │
                         │  Should improve       │
                         │  Leave it             │
                         │  ──────────────────   │
                         │  "Ship it." /         │
                         │  "Not yet: <one       │
                         │   thing in the way>"  │
                         └──────────────────────┘
```

Dad scales to the change. A typo gets a glance; a new auth flow gets the full panel. He's thorough when it matters, and thoroughness costs tokens. That's the point.

## What a verdict looks like

> **Fix before shipping**
> - `src/cache.ts`: you wrote a `CacheFactory` that builds exactly one kind of cache. It's a `Map` with extra steps and a diagram you have to hold in your head. Delete the factory. Use the `Map`.
> - `useTimer.ts:23`: the interval is never cleared on unmount. This leaks. It'll look fine in dev and it'll page someone at 3am.
>
> **Should improve**
> - `formatDate` reimplements what `Intl.DateTimeFormat` already does, including a bug on single-digit days. Not blocking. But you know better.
>
> **Leave it**
> - A reviewer flagged your four-line function as "not abstracted enough." Ignore it. It's four lines. It's perfect.
>
> **Not yet: clear that interval before anything else merges.**

No compliment sandwiches. If it's good, he says "Ship it." and he's done.

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
