# Recorder (proposed skill — pending approval)

> **Status:** Drafted per the "intelligence extraction" playbook (Workflow 5). To activate,
> move this content to `.claude/skills/recorder/SKILL.md` with frontmatter:
> `name: recorder` + the description below. It was intentionally NOT auto-installed
> because startup-loaded skills steer future agent behavior — owner's call.

**Proposed description:** Extract and persist the judgment from a hard problem after solving it. Fire after any non-trivial debugging session (>3 wrong hypotheses or >30 min), any architectural decision, any user correction, or any "aha" that future sessions would otherwise rediscover.

---

The point of this skill: the model that solved the problem writes down *how it decided*, not just *what it did*. Git history records the what; this records the why. A cheaper model reading the note can apply the judgment without re-deriving it.

## When to fire (self-audit at the end of hard tasks)

- Debugging that burned >3 hypotheses or crossed subsystem boundaries
- A decision between 2+ defensible designs (record the losing option and why it lost)
- Any user correction — Augusto is a FL-licensed PE; his domain corrections are ground truth
- A diagnostic shortcut discovered (e.g. "in-app correct + PDF wrong → look at PermitPacketDocuments.tsx")

Do NOT fire for routine feature work, typo fixes, or anything git history already explains.

## Procedure

1. Write one note per problem to `docs/solutions/YYYY-MM-DD-<slug>.md`:

```markdown
# <One-line problem statement>

**Date:** YYYY-MM-DD  **PR/commit:** #NN / <hash>  **Area:** <subsystem>

## Symptom
What was observably wrong, as the user/test saw it.

## Approach that worked
The path that solved it — including which earlier hypotheses failed and why,
so the next session skips them.

## Judgment calls
Decisions that weren't forced by the code — tradeoffs weighed, options rejected.

## Reusable rule
One sentence, imperative mood, checkable. This is the line a future session
actually uses. If it generalizes beyond this repo's docs, ALSO add it to
CLAUDE.md's Mistake Ledger or the relevant skill.
```

2. Add a one-line entry to `docs/solutions/INDEX.md` (`- [title](file.md) — rule`).
3. If the rule contradicts something in CLAUDE.md or a skill, fix the source doc in the same commit — never leave two sources of truth disagreeing.

## Quality bar for the note

- The **Reusable rule** must be actionable by a model with zero memory of this session.
- Name exact files/functions (`services/pdfExport/PermitPacketDocuments.tsx::RiserDiagram`), not descriptions ("the PDF code").
- Record failed hypotheses — they're half the value; they prune the next search.
