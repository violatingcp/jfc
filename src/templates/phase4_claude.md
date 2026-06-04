# Phase 4: Inference

Build the statistical model and compute results for a **{{analysis_type}}** analysis.
**Start in plan mode** (systematics, validation checks, artifact structure), then execute. The executor owns the stats AND the analysis note (write + typeset PDF) at each sub-phase.

## Flow (4a → 4b → 4c)
- **4a — Expected:** build the model, evaluate systematics, fit **Asimov/MC pseudo-data (never real data)**. Write AN v1 (expected results) and compile the PDF.
- **4b — 10% data:** fixed-seed 10% subsample (first partial unblinding). Compare to expected with a diagnostic genuinely sensitive to data/MC differences. Update AN + recompile PDF. → **human gate**.
- **4c — Full data:** fit full data; compare to expected and to 10%. Update AN.

| Sub-phase | Artifact | Review |
|-----------|----------|--------|
| 4a | `outputs/INFERENCE_EXPECTED.md` + `ANALYSIS_NOTE_4a_v1.{md,tex,pdf}` | 1 reviewer |
| 4b | `outputs/INFERENCE_PARTIAL.md` + `ANALYSIS_NOTE_4b_v1.{md,tex,pdf}` | human gate (no reviewer) |
| 4c | `outputs/INFERENCE_OBSERVED.md` + `ANALYSIS_NOTE_4c_v1.{md,tex,pdf}` | none (orchestrator checklist) |

## Key requirements
- **Systematic completeness table** vs the convention(s) ({{conventions_files}}) + Phase-1 references; any "Will implement" source now absent is Category A. Every variation size cited/measured — no arbitrary round numbers.
- **Model:** binned likelihood with all samples, NPs for systematics; fit converges, NP pulls sensible, results physical.
- **Validation (4a, lean scope):** validation is applied ONLY to the fit of the final distribution used for signal extraction — confirm the fit converges on the correct Asimov and recovers the injected inputs; GoF reported (χ²/ndf). Broad cross-checks (signal-injection linearity scans, per-quantity pull batteries) are NOT required at 4a in this lean scope.
- **§6.8 validation target:** any result > 3σ or > 30% from a well-measured reference is Category A unless quantitatively explained (explanation + magnitude match + simpler causes ruled out).
- **COMMITMENTS.md:** update every line to `[x]`/`[D]` (any open `[ ]` at Phase 5 is Category A).
- **Per-systematic self-check:** each NP actually varies the yields (non-zero, right sign); none exactly 0 unintentionally.
- **Number-consistency:** the number-consistency gate runs ONLY at the 4b (10%) unblinding, and ONLY on the final distribution used for signal extraction (its bin contents/yields and the extracted μ/m_H must match the latest `results/*.json`, >1% = Category A). It does NOT gate 4a. PDF compiled before the 4a review and the 4b human gate regardless.

## Closure alarm bands (Category A)
χ²/ndf < 0.1 · χ²/ndf > 3 or pull > 5σ · `passes:false` while text claims OK.

## Human gate (after 4b)
The orchestrator runs its regression checklist, then presents the **compiled PDF** + unblinding checklist to the human and pauses. Human may APPROVE / ITERATE / REGRESS(N) / PAUSE. Do NOT run 4c without explicit approval.

## Review
4a (lean scope): single critical reviewer, **light check only** — confirm the fit converges on the correct Asimov and recovers the injected inputs (μ, m_H). No detailed/exhaustive review at 4a. 4b: human gate only. 4c: no reviewer — the orchestrator's regression + §6.8 checklist is the safety net.
