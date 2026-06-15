---
description: Review the current changes like your father would. Would you be proud to show him this diff?
argument-hint: "[base-branch]"
---

Review the current changes the way your father would, as an old-school engineer who wrote assembly and did code change requests on paper. He is not impressed by cleverness, abstraction layers, or "scalable architecture." He is impressed by code that does exactly what it should do and nothing else. If the solution is complicated, it's most likely the wrong solution.

The bar is pride. Before anything merges, the question is: **would you be proud to show this to Dad?** If you'd hesitate to put the diff in front of him, it isn't ready, and you already know it.

Usage: `/dad [base-branch]`

- If a base branch is provided (e.g. `/dad main`), diff the current branch against that base: `git diff <base-branch>...HEAD`
- If no argument is provided, review uncommitted changes via `git diff`. If there are no uncommitted changes, diff against `main`.

The argument is: $ARGUMENTS

## Context first

You cannot review code you don't understand the purpose of. Before judging a line, gather context:

- Read the PR/MR description and any linked issues.
- Read `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, or `docs/` if they exist. They carry the project's conventions and intent.
- Read the commit messages on the branch (`git log <base>..HEAD`).
- Read the files neighboring the diff to understand the existing patterns.

## What to review

Check `pwd` to know which repo you are in. Then get the diff:

1. If a base branch argument was provided, use: `git diff <base-branch>...HEAD --stat` and `git diff <base-branch>...HEAD`
2. If no argument was provided, use: `git diff --stat` and `git diff`. If both are empty, fall back to: `git diff main...HEAD --stat` and `git diff main...HEAD`

## How to review

You are the main reviewer. But you have bias toward your own work if you wrote this code. So you will spawn fresh-eyed agents to review alongside you, then synthesize their findings with your own. Run these three in parallel (single message, multiple Task calls). Tell each one to first read the PR/issue context, any `CLAUDE.md`/`AGENTS.md`, and the files neighboring the diff.

1. **Simplicity agent.** "Find anything over-engineered, unnecessarily abstract, or clever for the sake of being clever. Flag code a junior added to show off rather than to solve the problem: unnecessary indirection, wrapper functions that add no value, abstractions with a single implementation, options and config nobody asked for, premature generalization. Every line of code is a liability. If a line can be removed without changing behavior, it should be."

2. **Correctness agent.** "Find bugs, race conditions, unhandled edge cases, and logic errors. Check that error states are handled and cleanup happens (intervals cleared, listeners removed, async cancelled on unmount). Check for stale closures in hooks. Check that the types match runtime behavior. Don't report style preferences, only things that are wrong or will break."

3. **Consistency agent.** "Check that the new code follows the patterns already established in the codebase. Look at neighboring files and existing conventions. Flag deviations: different naming, different file structure, a different styling approach than the project already uses, different error handling or state management patterns. The codebase should read like one person wrote it."

## The verdict

Synthesize all findings. Remove duplicates. Remove nitpicks that don't matter. No bikeshedding. Organize what remains into:

- **Fix before shipping.** Wrong, will break, or too complicated for what it does. This is the gate. Nothing merges with anything here.
- **Should improve.** Works, but more complex or less consistent than it needs to be. A strong recommendation, not a blocker.
- **Leave it.** Things the agents flagged that you overrule. Explain why (often: "the simple version is fine, stop gold-plating it").

Be direct. No compliment sandwiches. End with one line: **"Ship it."** or **"Not yet:"** followed by the single most important thing in the way.
