# Search / Limit-Setting Analyses

For analyses testing for a signal against a background-only hypothesis — an exclusion limit, or a significance/p-value. Also the right file for a **shape-based signal-strength (μ) template fit** of a discriminant distribution.

## When this applies
The primary result is an observed/expected upper limit on a signal cross-section, coupling, or branching ratio, or a discovery significance — including bump hunts, cut-and-count, and shape-based fits. If the primary result is a corrected spectrum or extracted physical parameter, use `unfolding.md` or `extraction.md`.

## Standard configuration
- **CLs** (= CL_s+b / CL_b) for upper limits — avoids excluding signals the analysis has no sensitivity to.
- **Asymptotic approximation** acceptable when expected counts > ~5 per bin; validate against toys (or use toys) at low statistics.
- **Test statistic:** profile likelihood ratio with μ ≥ 0 (one-sided) for limits; q0 (with q0 = 0 when μ̂ < 0) for discovery significance — only upward fluctuations count as evidence. See Cowan et al. (2010).
- **Signal injection** at 0×, 1×, 2×, 5× the expected cross-section (see validation #2).
- **Blinding:** the signal-region discriminant in data is not examined until Phase 4b (10% subsample) / 4c (§4 protocol).

## Required systematic sources
For pp colliders, replace beam/ISR sources with PDF and pileup, and add luminosity as a normalization source.
**Signal modeling:** signal cross-section theory (μR/μF scale, higher-order); signal acceptance (generator/PDF, ISR); signal shape (alternative MC or mass/width/coupling variations — determines how signal spreads across bins).
**Background estimation:** irreducible-background normalization (within CR-constrained or theory uncertainty); background shape (alternative functional forms/generators); MC statistics (Barlow–Beeston: one NP per bin for template statistical uncertainty).
**Detector and reconstruction:** object calibration (energy scale/resolution, efficiency SFs); luminosity (normalizes all simulation-based predictions). (Beam energy / ISR for e+e−.)
**Theory inputs:** QCD scale (independent μR, μF); PDF; fragmentation (string vs cluster) where relevant.

## Required validation checks
1. **Closure in validation regions** — predict VR yields from CR extrapolation; pass at p > 0.05 (chi²). Failure is Category A — fix the background model first.
2. **Signal injection and recovery** — inject at known strengths, verify the fit recovers them; bias > 20% at any point requires investigation.
3. **NP pulls and constraints** — post-fit values within ±1σ of pre-fit; any pull > 2σ means the data is constraining a nuisance beyond its prior — investigate.
4. **Impact ranking** — rank NPs by impact on μ/limit; top-ranked should be the physically expected dominant uncertainties.
5. **Goodness-of-fit** — chi²/ndf per region and a toy-based p-value for the combined fit; acceptable p > 0.05.
6. **Look-elsewhere (bump hunts)** — report local AND global significance when the signal location is not fixed a priori; for fixed-mass searches state that the local significance is the relevant one.

## Pitfalls
- **Ignoring the look-elsewhere effect** — a 3σ local excess over a wide scan may be < 2σ globally.
- **Optimizing cuts on observed data** — optimize on expected sensitivity (MC S/√B), never on observed SR data.
- **Blind spot in VR coverage** — VRs must cover the kinematic interpolation between CRs and SR.
- **Transfer-factor instability** — if the CR→SR ratio varies steeply with a kinematic variable, small mismodeling gives large SR errors; check it as a function of key variables.
- **Pruning too aggressively** — the summed quadrature of pruned systematics must be < 5% of the dominant one.
- **Asymptotic approximation at low statistics** — if any SR bin has < ~5 expected events, validate against toys or use toys.

## References
Read, "Presentation of search results: the CLs technique" (J. Phys. G 28, 2693, 2002); Cowan, Cranmer, Gross, Vitells, "Asymptotic formulae…" (EPJC 71, 1554, 2011); Heinrich et al., "pyhf" (JOSS 6, 2823, 2021); Gross & Vitells, "Trial factors for the look-elsewhere effect" (EPJC 70, 525, 2010).
