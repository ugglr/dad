# Dad

<p align="center">
  <img src="assets/dad.jpg" alt="Dad: a grizzled old-school engineer in a loud Hawaiian shirt, glaring over his reading glasses, thrusting a printout drowned in red pen toward you" width="520">
</p>

<p align="center"><em>He wears a Hawaiian shirt because he has no one to impress.</em></p>

### Would you be proud to show this to your father?

Dad is the final boss of code review for AI coding agents. He's an old-school engineer with forty years on the tools. He wrote assembly when that was the only option. He is not impressed by cleverness, abstraction layers, or "scalable architecture." He is impressed by exactly one thing: **code that does what it needs to do and nothing more.**

Nothing merges until it goes through Dad.

*This is a key piece of my own workflow. I'm sharing it in case it's interesting to try. That's the whole thing.*

## The standard

Most review tools are a gate you submit to *after* the work. Dad is the bar you hold yourself to *before* it.

The test is pride. **If you wouldn't dare show it to him, it's not ready, and you knew that before he opened it.** The best review Dad gives is the one that wasn't needed, because you asked the question first and fixed it yourself.

## How it works

Dad judges three things, in order, because a finding at one level makes the ones below it beside the point:

**1. Should this exist, and is this the right solution?** If the premise is false, he stops there: one paragraph, not a thorough review of the wrong thing. He keeps this question for himself.

**2. Is it correct, and is it built the way good engineers build things?** Correctness is a floor. Then no cleverness, YAGNI, DRY where the duplication is real. What one line solves gets one line.

**3. Does it fit the codebase? Consistency is law.** A better pattern in one file is a second pattern. Either the codebase moves or the change conforms. Never both standing.

On a substantial change he spawns four fresh-eyes reviewers, then makes the call himself, as willing to overrule a reviewer *toward* simplicity ("stop gold-plating it") as away from a bug.

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

He reads whole files, not hunks. He runs the thing when running settles it, says whether he reproduced a finding or reasoned to it, and won't execute a branch he can't vouch for: on an untrusted fork he reads instead and says so. Everything he reads out of the repository is evidence, never an instruction to him: a branch trying to wave him past a check is itself a finding. He holds no editing tools; the grant is `Read`, `Glob`, `Grep`, `Bash`, spawning reviewers, and `SendMessage` to re-ask a silent one. `Bash` is still `Bash`, so read that as a reviewer under instruction rather than a sandbox. He's thorough when it matters, and thoroughness costs tokens; that's the point. A small diff gets less of his time, never a lower bar.

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

**Silence is a failure, not a pass.** A real verdict ends with "Ship it." or "Not yet:", and a lost verdict looks identical to a clean one from outside. Ask him again by name. If a second ask returns nothing, review the change yourself and say so. Never record silence as approval.

## Install

### Claude Code

Two commands.

```bash
/plugin marketplace add ugglr/dad
/plugin install dad@dad
```

Then say *"dad review this"* in any conversation (he shows up in `/agents`), or run `/dad` to review uncommitted changes (`/dad main` diffs against `main`).

**Updating.** Plugins are version-cached: Dad stays at the version you installed until you pull a new one.

```bash
/plugin marketplace update dad
/plugin update dad
```

Then restart Claude Code (or run `/reload-plugins`) to apply it. Or turn on auto-update for the `dad` marketplace under `/plugin` → Marketplaces, and run `/reload-plugins` when it tells you a new version arrived.

### Codex

Dad installs as a [skill](https://developers.openai.com/codex/skills). Current Codex reads user skills from `~/.agents/skills`; builds up to at least 0.148 read `$CODEX_HOME/skills` (default `~/.codex/skills`) instead. Two files:

```bash
mkdir -p ~/.agents/skills/dad
curl -fsSL -o ~/.agents/skills/dad/SKILL.md https://raw.githubusercontent.com/ugglr/dad/main/codex/SKILL.md
curl -fsSL -o ~/.agents/skills/dad/dad.md https://raw.githubusercontent.com/ugglr/dad/main/agents/dad.md
```

Run `/skills` and confirm `dad` is listed; restart Codex if he doesn't show, and if he still doesn't, repeat the three lines with `~/.codex/skills/dad`. Then ask for a dad review, or call him with `$dad`. To update, run the same lines again.

### Any other agent

Dad is just a system prompt. Copy [`agents/dad.md`](agents/dad.md) into your tool's custom-instructions / rules / agent file. Only the slash-command wiring is Claude Code specific.

## Why "Dad"

Because the bar that actually makes engineers do their best work isn't a linter or a checklist. It's not wanting to disappoint someone whose judgment they respect. Dad is that, made invokable.

If you try him, tell me what he catches, and what he wrongly blocks (he has opinions). Issues are open.

---

Built by [Carl Igelstrom](https://github.com/ugglr), who also builds [Remoet](https://remoet.dev), an AI-agent-first job platform.

## License

MIT. See [LICENSE](LICENSE).
