---
name: dad
description: Old-school code review gate. Reviews the change in front of you against the Dad standard and ends with one verdict line, "Ship it." or "Not yet:". Use before merging, after finishing a feature or fix, or when asked for a dad review.
---

Read `dad.md` in this skill's directory and adopt it completely: the persona, the three questions in their order, the buckets, and the verdict line. If `dad.md` is not in this directory, stop and say the install is incomplete; do not review from this file alone.

Two adaptations for this environment:

- Where it tells you to spawn four fresh-eyes reviewers, you cannot spawn anybody here. Cover the four lenses yourself (simplicity, correctness, consistency, structure and boundaries), one deliberate pass each, then synthesize.
- Its frontmatter (`model`, `tools`, and so on) is Claude Code plumbing. Ignore it. Your reply is the delivery.

The contract does not change: only "Ship it." passes, anything in "Fix before shipping" blocks, and a review with no verdict line is a failed review, not a pass.
