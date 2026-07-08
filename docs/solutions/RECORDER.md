# extract-approach (proposed skill — pending approval)

> **Status:** Drafted per the "Do this on your last day with Fable" playbook (Move 5, verified
> against the full article 2026-07-08). The article calls this **the move to install FIRST** —
> it compounds: every hard problem solved afterward leaves its reasoning behind automatically.
> To activate: (1) move the skill block below to `.claude/skills/extract-approach/SKILL.md`,
> and (2) add the **learning law** section below to CLAUDE.md so it fires without being asked.
> It was intentionally NOT auto-installed — startup-loaded skills steer future agent behavior,
> so activation is the owner's call.

## The skill (→ `.claude/skills/extract-approach/SKILL.md`)

```markdown
---
name: extract-approach
description: After solving any non-trivial problem, document the approach so a less capable model can replicate the thinking. Triggers after debugging, architecture decisions, tricky builds, or any solution that took real reasoning.
---

# extract-approach

After solving a hard problem, write ONE note to `docs/solutions/YYYY-MM-DD-<slug>.md`:

- **The problem**, in one line
- **The approach**: how the problem was decomposed, in plain steps — including which
  hypotheses failed and why, so the next session skips them
- **The judgment calls**: what was deliberately NOT done, and why (options weighed,
  tradeoffs rejected)
- **The reusable rule**: the one-line principle a future model should apply when it
  smells a similar problem — imperative mood, checkable

Keep it under a page. Write it so a weaker model reading it cold could follow the same path.

Then:
1. Add a one-line entry to `docs/solutions/INDEX.md` (`- [title](file.md) — rule`).
2. If the rule contradicts CLAUDE.md or a skill, fix that source doc in the same
   commit — never leave two sources of truth disagreeing. If it generalizes, add a
   row to CLAUDE.md's Mistake Ledger.

## When to fire (self-audit at the end of hard tasks)

- Debugging that burned >3 hypotheses or crossed subsystem boundaries
- A decision between 2+ defensible designs (record the losing option and why it lost)
- Any user correction — Augusto is a FL-licensed PE; his domain corrections are ground truth
- A diagnostic shortcut discovered (e.g. "in-app correct + PDF wrong → look at
  services/pdfExport/PermitPacketDocuments.tsx")

Do NOT fire for routine feature work, typo fixes, or anything git history already explains.

## Quality bar for the note

- The reusable rule must be actionable by a model with zero memory of this session.
- Name exact files/functions, not descriptions ("the PDF code").
- Failed hypotheses are half the value — they prune the next search.
```

## The CLAUDE.md wiring (→ add as a section in CLAUDE.md on activation)

```markdown
## Learning Law

After every non-trivial solved problem, run the extract-approach skill before moving on.
A solution without its learnings note is unfinished work.
```

*Note: destination adapted from the article's `learnings/<date>-<slug>.md` to this repo's
`docs/solutions/` (INDEX.md already lives there). Everything else follows the article's spec.*
