# Phase 1: Strategy

Develop the strategy for a **{{analysis_type}}** analysis.
**Start in plan mode** (plan literature, samples, artifact structure), then execute.

## Output artifact
`outputs/STRATEGY.md` — motivation, sample inventory, selection approach, systematic plan, technique selection.

## Key requirements
- Physics motivation and the discriminating observable(s).
- Sample inventory (data + MC); backgrounds classified (irreducible / reducible / instrumental).
- ≥2 qualitatively different selection approaches (one MVA-based, or document why not with a [D] label).
- Systematic plan: enumerate every source in the applicable convention(s) ({{conventions_files}}) as "Will implement" or "Not applicable because…". Each size tied to a citable/measured value — no arbitrary inflations.
- Reference table: 2–3 published analyses with their systematic programs.
- Technique selection (this determines which convention applies downstream).
- Every numeric constant cited (RAG / web / paper) — never from memory.
- Query the experiment corpus (RAG) if available; cite sources. Define [A]/[L]/[D] labels.

## Self-check
- [ ] ≥2 distinct approaches; MVA used or infeasibility documented [D].
- [ ] Systematic plan covers every convention source (implement / N-A).
- [ ] Reference table present; technique chosen and justified; all constants cited.

## Review
Single critical reviewer (`agents/critical_reviewer.md`) — PASS / ITERATE; only Category A blocks. Findings to `review/critical/`.
