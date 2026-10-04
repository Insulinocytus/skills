---
name: distill-pr
description: "Distill why a PR exists: problem, origin, approach, trade-offs, gaps. Use when reading a PR, or asked why a PR or change exists."
---

Pull the signal out of a PR whose body and linked ticket are padded with AI-written prose. Prose restates; the diff and the humans carry the facts.

## Evidence ranking

Rank sources by trust, highest first:

1. The diff.
2. Commit messages and git history: merges, reverts, what landed on the base meanwhile.
3. Human-written comments: PR review threads, ticket comments and discussions.
4. Ticket body.
5. PR body.

Read 4 and 5 as claims to check against 1–3. A single human comment is often the real origin that pages of ticket prose bury. Where a higher source contradicts a lower one, the higher wins and the contradiction goes under **Gaps**.

## Steps

1. **Gather.** Fetch the PR body, diff, commits, and review comments, plus every ticket or issue it links with that ticket's comments, using whatever tools this environment offers. Done when you hold each source or know it is unreachable.
2. **Trace.** For each report field still open, go one hop out: anything the gathered sources name, by link or bare ID (a parent or predecessor ticket, a related PR), plus recent commits on the touched files. One hop, then stop. Anything still open is either an `[inference]` with its evidence, or a gap.
3. **Report**, in the format below.

## Report

Five fields, 2–3 short sentences each, plain words; the whole report, verdict included, fits in 15 lines. Tag each claim with its source: `(diff)`, `(commit)`, `(comment)`, `(ticket)`, `(PR body)`, or `[inference]`.

- **Problem**: what goes wrong, and for whom, without this PR.
- **Origin**: who raised it, and where: a prior ticket, a review comment, a bug, an incident.
- **Approach**: the mechanism the diff actually uses, told the way a reviewer would explain it aloud.
- **Trade-offs**: what was given up, deferred, or chosen over what.
- **Gaps**: questions neither the PR nor the ticket answers, and every place the prose contradicts the code.

Close with one line, set apart from the facts:

- **Verdict**: does the PR earn its existence, weighed against the volume of its prose? Name the parts that are justified and the parts resting on a thin reason.
