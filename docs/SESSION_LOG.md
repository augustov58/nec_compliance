# Session Log

**Purpose**: Tracks recent work for seamless handoff between Claude instances.
**Maintenance Rule**: Keep only the last 2 sessions. At the start of a new session, delete older entries — git history preserves everything.

**Last Updated**: 2026-07-09

---

### Session: 2026-07-07 → 2026-07-09 — Fable intelligence extraction + NEC 220.87(2) measured-demand fix (PR #118 merged, squash `44ca8bc`)

**Focus**: Applied the "Do this on your last day with Fable" playbook (Machina, X 2026-07-06) in a worktree: rewrote CLAUDE.md as an explicit operating manual (Mistake Ledger, Escalation Rules, hardened Verification Protocol, skills index), wrote 3 subsystem skills (`nec-calc-service`, `permit-packet`, `packet-verification`), and scaffolded `docs/solutions/`. Mid-work, verified the source article against the live rendered X page — the saved Obsidian note contained four fabricated passages from a bogus "X API verbatim" recovery; corrected the note and reconciled the repo artifacts (added the missing Escalation Rules element; aligned the recorder draft to the article's `extract-approach` spec, transcribed from an image embed).

**The big one — NEC 220.87(2) fix (PE review finding)**: During PR review Augusto flagged that measured demand should take the 125% factor. Confirmed against code text: NEC 220.87(2) requires "the maximum demand at 125 percent plus the new load" ≤ service rating. All three implementation sites (`serviceUpgrade.ts` × 2 functions, `PermitPacketDocuments.tsx::NEC22087NarrativePage`, `multiFamilyEV.ts` PATH A) exempted `utility_bill`/`load_study` — understating existing load, the UNSAFE direction (the packet page even printed the correct condition-2 header above math that omitted the factor). Fixed all three + `ServiceUpgradeWizard` copy (which carried labels from two stale eras); the 2026-05-27 ruling stands (`calculated` → direct; `manual` → ×1.25 defensive). 4 tests rewritten to the corrected rule; 1035 passing; tsc clean. Visual proof: rendered the narrative page through the real @react-pdf pipeline — the money case (measured 120 + 48 new on 192 kVA capacity) now correctly FAILS at 103.1% where the old code approved it at "168 ≤ 192". PDFs at `example_reports/NEC22087_Narrative_*_PR118_2026-07-09.pdf`.

**Process lessons**:
- **The two 220.87 bugs were mirror images with different failure loudness.** Over-applying a factor fails loud (388% utilization gets questioned); under-applying fails silent (packet looks fine, margin isn't there). Direction-of-failure is now recorded in the Mistake Ledger and `docs/solutions/2026-07-09-nec22087-measured-demand-125pct.md`.
- **"Already solved" ≠ solved.** Sprint 2 settled only the *calculated* half of 220.87; the measured half survived two reviews because the pre-existing code already exempted it. When a value is exempted from a code-mandated factor because it's "already X", verify against the code section's literal text.
- **Test assertions that only check `blob.size > 0` prove nothing about correctness** — `nec22087NarrativePdf.test.ts` stayed green through the behavior change. Math assertions live in `calculations-extended.test.ts`; render checked visually.
- **Browser topology**: claude-in-chrome = Augusto's remote browser via Orca IDE (no localhost access to this host); Playwright MCP = local browser for dev servers; e2e auth via user-run `tests/e2e/auth.setup.ts` (env-var credentials, never handled by the agent). Saved to memory as `environment_browser_topology`.

**Post-merge housekeeping (this entry)**: worktree + branches removed; `extract-approach` skill ACTIVATED (`.claude/skills/extract-approach/SKILL.md` + Learning Law in CLAUDE.md); CHANGELOG + SESSION_LOG updated.

**Next steps**: (1) optional in-app e2e pass of the Service Upgrade Wizard once Augusto refreshes the Playwright auth state; (2) the article's remaining runtime workflows on request — consultant audit (repo + business → executable roadmap), second-brain research runs into Obsidian.

---

### Session: 2026-05-23 / 2026-05-24 — Post-Sprint-2C panel-PDF demand surfacing + type baseline + CI gate (PRs #94, #95, #96, #97, #98, #99 all merged)

**Focus**: Two-day arc closing three threads deferred during Sprint 2C. Day 1 (2026-05-23): cleaned the TypeScript baseline (447 → 0 errors), added the first CI workflow in the repo with a `tsc --noEmit` gate, regenerated `database.types.ts` from the Supabase MCP. Day 2 (2026-05-24): three layered panel-schedule PDF improvements driven by user visual review of multifamily + commercial generated packets — surface NEC demand kVA/amps inline on the Load Summary, add a per-load-type NEC 220 audit breakdown table, and route the dwelling card's NEC article (220.82 vs 220.83) by construction context.

**Status**: 6 PRs merged sequentially. Build green. 995/995 tests passing throughout (zero net test count change — all production code, no test additions other than the two opt-in PDF disk-dump blocks in `permitPacketE2E.test.ts`).

**Architecture decisions worth carrying forward**:

- **Trust-the-aggregator pattern beats reimplementing NEC math at every render site.** The panel-schedule PDF doesn't redo any NEC 220 cascade math — it asks `calculateAggregatedLoad` (the single source of truth for the in-app Panel Summary) and trusts the result. The `necReferences` array on the result drives the inline article annotation in the card title; the `demandBreakdown` array drives the audit table. If a future NEC update changes how the aggregator computes demand, the PDF automatically picks up the new behavior without renderer changes. Replicable pattern for any rendered NEC artifact: the calc service is the source of truth, the renderer is dumb-but-structured.
- **The per-row tightening is the load-bearing fix when fighting tabular overflow.** PR #98's third sub-change recovered ~80pt total by tightening panel-schedule spacing. One-time fixes (header marginBottom 20→8, paddingBottom 10→4, infoRow marginBottom 3→1, tableContainer margins, etc.) saved ~48pt. But the load-bearing change was `tableRow.paddingVertical 4→2.5` × 21 rows = 31.5pt — the multiplied-cost source. **Lesson for future overflow fights: identify the multiplied-cost line first; fixed-cost savings rarely accumulate enough alone.**
- **Reuse existing semantic props rather than introducing new ones with overlapping meaning.** PR #99 needed a signal for "is this existing construction or new" to route between NEC 220.82 and 220.83. `showExistingNewMarkers` was already plumbed for the "* = Proposed new circuit" decorator from Sprint 2C. Same boolean semantically, two distinct uses. No new prop. **General rule**: when an upstream signal already encodes the answer to a downstream question, route off it directly even if the original use was visual-only.
- **Postgres enums > CHECK constraints when narrow-typed columns matter.** PR #96's regen surfaced that `cover_mode` widened from a 3-literal union to `string` because Supabase's gen-types doesn't introspect CHECK constraints — the runtime safety is there, but the type system can't see it. Fixed inline with `as CoverMode` casts at two call sites, but the deeper lesson is for the next narrow-typed text column: use `CREATE TYPE x AS ENUM (...)` so gen-types can narrow it. Already at least one such column on the schema worth migrating; documented as a low-stakes high-leverage cleanup for a future session.
- **The audit-breakdown pattern is a data → UI evolution worth replicating.** Inspector ergonomics shifted measurably: an AHJ reviewer now sees the NEC 220 cascade application on the same page as the panel schedule, eliminating the cross-reference flip to the separate Load Calculation Summary. The breakdown table's compact 5-col layout (`Load Type | Connected | Demand | Factor | NEC`) fits below a 42-row schedule with ~30% page room to spare. **Same pattern applies anywhere we have a "summary number that was derived from a multi-rule cascade"** — short circuit AIC, voltage drop %, conductor sizing, etc. Each renders today as a single result number with no per-rule visibility. The breakdown pattern is the next polish layer.

**Process gotchas worth remembering**:

- **Vite skipping type validation is a real production hazard.** PR #94 surfaced one real bug among 447 type errors: `MultiFamilyEVInput.evChargersPerUnit` was a stale field that Vite's transpile-only build had silently shipped past the broken type checker. Without PR #95's CI gate, the 0-error baseline would have drifted right back up. **For any new repo / monorepo additions: include `npx tsc --noEmit` in CI from day one.**
- **The Supabase MCP `generate_typescript_types` works identically to the CLI but without local auth state.** PR #96 used `mcp__plugin_supabase_supabase__generate_typescript_types({project_id: "..."})` — output is byte-identical to `supabase gen types typescript --project-id` from the CLI. For future regens, either path works; the MCP path is one tool invocation if already authenticated, the CLI path is one shell line. No need to lean on one over the other.
- **Visual review (PDFs converted to PNG + sent via SendUserFile) is a real bug-finding tool.** User caught the dwelling false-positive on commercial MDPs (kitchen-shape circuits triggering single-family-MDP detection) on a Commercial Building generated packet — would have shipped uncaught without the visual review cycle. Same cycle caught the 42-circuit overflow in PR #98 and led to the spacing-tightening third sub-change. **Pattern: send each generated PDF as a SendUserFile artifact (status: normal) so the user opens it natively; user names a specific visual issue → renderer site located via grep → minimal targeted edit → regenerate + re-send → user confirms or names the next thing.** Scales far better than trying to anticipate every visual detail upfront.
- **Fold related fixes into the open PR, not as land-and-followup.** Mid-PR-#98 review surfaced the layout overflow on 42-circuit panels (NOT a regression — PR #97 was clean, but #98's added breakdown row pushed it over). Folded the tightening into the same PR per the user's saved preference (`feedback_scope_during_review.md`) rather than opening a separate PR. Three commits in one PR vs three separate PRs is the right call when the fixes are in the same architectural area.

**Deliverables**:

| PR | Branch | Files | Result |
|---|---|---|---|
| **#94** | `fix/types-baseline` | many files, 447→0 tsc errors | ✅ Merged 2026-05-23 |
| **#95** | `ci/add-tsc-gate` | `.github/workflows/ci.yml` (new) | ✅ Merged 2026-05-23 |
| **#96** | `chore/regen-database-types` | `lib/database.types.ts` + 2 cover_mode call sites | ✅ Merged 2026-05-23 |
| **#97** | `fix/panel-pdf-demand-display` | 4 files, +321 / -98 LOC | ✅ Merged 2026-05-23 |
| **#98** | `fix/panel-pdf-commercial-demand` | 4 files, +190 / -29 LOC across 2 commits | ✅ Merged 2026-05-24 |
| **#99** | `fix/panel-pdf-dwelling-breakdown` | 2 files, +188 / -49 LOC | ✅ Merged 2026-05-24 |

**No DB migrations.** All six PRs are pure-code: type cleanup, CI infrastructure, types regen, and PDF rendering changes. Backward compat preserved everywhere.

**Visual proofs in `example_reports/`** (regeneratable via `E2E_DUMP_PDFS=1 npm test`):
- `Permit_Packet_MF_EV_Existing_2026-05-17.pdf` — multifamily packet with MDP + H1 sub-panel pages showing the new aggregator card + breakdown table
- `PanelSchedule_Dwelling_Existing_22083.pdf` — single-page dwelling MDP showing the NEC 220.83 card + tier-split breakdown
- `PanelSchedule_Dwelling_New_22082.pdf` — same fixture rendered as new-construction, NEC 220.82 routing

**Follow-ups not on this session's queue**:
1. **STRATEGIC_ANALYSIS.md + DISTRIBUTION_PLAYBOOK.md feature-inventory sweep** — still owed from the prior session; now also needs the audit-breakdown story added as a competitive differentiator (no competing SaaS surfaces the NEC 220 cascade on the panel page itself).
2. **Commercial demand-factor coverage extension** — `calculateAggregatedLoad`'s per-load-type helpers cover Receptacles / Motors / HVAC / Range / Dryer. Specialty commercial loads (welders @ NEC 220.55, restaurant kitchen equipment @ 220.56) currently bucket into "Other @ 100%" — not wrong but conservative. Extend coverage if a real packet surfaces it.
3. **Migrate CHECK-constrained text columns to Postgres enums** — surfaced by PR #96. `cover_mode` is the known one; grep the schema for other narrow-typed text columns with CHECK constraints and migrate them so the next gen-types regen produces narrow unions automatically.
4. **Apply audit-breakdown pattern to other multi-rule results** — short circuit AIC, voltage drop, conductor sizing all currently render as single numbers without per-rule visibility. Same pattern would surface those derivations on their respective PDF pages.

---

<!-- Earlier sessions (2026-05-17 Sprint 2C M3 Existing/New construction PRs #79/#80/#81, 2026-05-16 Sprint 2C M2 AHJ expansion PRs #70–#75, 2026-05-15 docs cleanup + JurisdictionRequirements font hotfix + dwelling-load follow-up PRs #63/#64/#65, 2026-05-13 Sprint 2B PR-4 Orlando manifest scaffold + AHJ-aware visibility, 2026-05-12 Sprint 2B PR-3 merge engine, 2026-05-10 Sprint 2A final 2 PRs, 2026-05-09 contractor-pivot + T&M Phase 1) rotated out per "keep last 2 sessions" rule. Git history preserves them. -->
