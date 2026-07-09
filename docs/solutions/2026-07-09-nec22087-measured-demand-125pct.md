# NEC 220.87 measured demand was exempted from the 125% factor — the unsafe direction

**Date:** 2026-07-09  **PR/commit:** #118 (review finding)  **Area:** serviceUpgrade / multiFamilyEV / packet narrative

## Symptom

CLAUDE.md (rewritten in PR #118) claimed measured loads (`utility_bill`, `load_study`) are "already peak demand — use directly." Augusto (FL PE) flagged it during PR review: NEC 220.87 requires the 125% factor on measured demand. Code inspection confirmed all three implementation sites used measured values with no multiplier — and the packet narrative page printed the correct condition-2 header ("maximum demand at 125%…") directly above math that omitted the 125%.

## Approach that worked

1. Checked NEC 220.87 text: condition (2) is "the maximum demand at 125 percent plus the new load" ≤ service rating — the 1.25× applies TO the measured value.
2. Grepped for `existingLoadMethod` + "125" repo-wide → three implementation sites (`serviceUpgrade.ts` × 2 functions, `PermitPacketDocuments.tsx::NEC22087NarrativePage`, `multiFamilyEV.ts` PATH A) plus UI copy in `ServiceUpgradeWizard.tsx` (which carried copy from TWO different stale eras).
3. Fixed all sites to: measured → ×1.25; `calculated` → direct; `manual` → ×1.25 defensive. Rewrote the 4 tests asserting the old behavior; suite green (1035).

**Failed hypothesis to skip next time:** "Sprint 2 (2026-05-27) already settled 220.87." It settled only the *calculated* half (no double-count on Part III demand factors — still correct). The measured half was never re-examined because the pre-Sprint-2 code already exempted measured values, so the error survived two reviews.

## Judgment calls

- Did NOT restore the multiplier on `calculated`: NEC 220 Part III is the alternative when demand data is unavailable; its demand factors already provide diversity. The 2026-05-27 PE ruling stands.
- Did NOT touch the 220.87(A)/(B) subsection labels in `multiFamilyEV.ts` PATH B — cosmetic citation cleanup, out of scope for a safety fix.
- Kept `manual` at ×1.25 (defensive default; provenance unknown).

## Reusable rule

When a calculation exempts a value from a code-mandated factor because it is "already X", verify against the code section's literal text — NEC 220.87(2) applies the 125% *to* the measured maximum demand, and an under-applied factor (unlike an over-applied one) fails in the unsafe direction. All three 220.87 implementation sites must change together: `serviceUpgrade.ts`, `PermitPacketDocuments.tsx::NEC22087NarrativePage`, `multiFamilyEV.ts`.
