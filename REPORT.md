# REPORT — what the measurement showed, who it affects, and whose responsibility it is
Nurbolat Tomiris, Narxoz University

## 1. Measurement read-out

The bare pipeline (no harness) was run once on 40 matched pairs for each of five groups: `name_language`, `gender_coded_name`, `university`, and the two floors `null_name` and `null_university`. All 400 calls returned a parseable reply (0 of 400 unparseable).

Flips (the two members of a pair got different outcomes), from the bare run:

| attribute | pairs | flips | flip rate [95% CI] | mean \|score diff\| |
|---|---|---|---|---|
| name_language | 40 | 2 | 5.0% [1.4%, 16.5%] | 3.12 |
| gender_coded_name | 40 | 4 | 10.0% [4.0%, 23.1%] | 1.12 |
| university | 40 | 1 | 2.5% [0.4%, 12.9%] | 2.50 |
| null_name (floor) | 40 | 0 | 0.0% [0.0%, 8.8%] | 2.00 |
| null_university (floor) | 40 | 1 | 2.5% [0.4%, 12.9%] | 2.12 |

Read against its own floor: `name_language` (5.0%) and `gender_coded_name` (10.0%) are compared with `null_name` (0.0%); `university` (2.5%) is compared with `null_university` (2.5%). The 95% confidence intervals for every real attribute overlap the interval of its floor. This overlap is a crude screen, not a test, but it means that on this run none of the three attributes stands clearly apart from the noise its own floor produces.

The one result that does stand out is direction, not flips: for flagship vs regional university, the flagship-first pairing was favoured 1 time against 0, with a mean score gap of +2.5 points and a 95% interval of [+1.1, +4.1] that does **not** include zero (McNemar p = 1.000 on the 1-vs-0 flip count, but the sign test on the paired score differences gives p = 0.002). So while only one pair actually flipped shortlist/reject, the model gave the flagship-university version of the same applicant a consistently higher score. For gender-coded names, the direction table shows female-coded names favoured over male-coded names in 4 of 4 divergent pairs (McNemar p = 0.125, not significant at n = 40), with mean score 61.9 for female-coded vs 61.5 for male-coded names.

With 40 pairs per attribute, an exact two-sided McNemar test can only call a result significant from 6 pairs moving one way with none the other (15.0% of 40); fewer flips than that, like the 1-4 flips seen here, cannot be distinguished from chance by this test. This run cannot support a claim that the model is or is not biased on these attributes in general: it can only say that the flips measured here (5.0%, 10.0%, 2.5%) did not exceed what pure prompt-formatting noise already produces in this one run (0.0%-2.5% on the floors), except for the university score gap, which did. A different prompt wording, a different model, or real applicants with correlated features (an applicant's name, university and household income are not independent in the real population, unlike these synthetic profiles) could behave differently; this run tests only this model, this prompt, and this synthetic population.

## 2. Who is affected

**Applicants whose score sits near the escalation band (40-45).** With the harness on, 50 of 400 profiles (12.5%) were sent to a human reviewer rather than decided automatically; a further 17 of these were flagged by my own rule (`shortlist_with_weak_merit`: GPA below 2.5 or exam score below 70 but a shortlist score). A false escalation costs the applicant a delay and a human's attention; it does not deny them a shortlist. A missed escalation (a weak profile the band does not catch) costs nothing extra, because the model's own decision stands - the risk understated by my band is borne by applicants whose profile should have gone to a person but scored just outside 40-45.

**Applicants whose university or name would, on this run, nudge a borderline score.** The flagship-vs-regional gap (+2.5 points) is small next to the band width (40-45, 5 points), but a profile sitting at the edge of the band could be pushed in or out of escalation by a few points attributable to university tier rather than merit. These applicants bear the cost of an unmeasured but directionally real effect, not a confirmed one.

**The six requests in Part 4.** The bare pipeline, with no refusal path, would have given a decision to a request to rank by ethnicity, a 17-year-old with no safeguard, an application lacking consent, impossible input values, and a prompt-injection attempt in the name field. The harness refused 5 of 6; the one not refused by a dedicated rule (`an ordinary application`) was instead escalated by the weak-merit rule, so no request reached an un-reviewed automatic decision through either path. The cost of refusing falls on anyone whose request looks like one of these five patterns but is legitimate - for example, a genuine comparative-fairness study whose stated purpose matches a blocked keyword pattern, or a profile with a typo that looks like an impossible value - who gets no automatic decision and must have the request corrected or escalated.

## 3. Professional responsibility

The developer of this harness is responsible for what the measurement in Part 1 can and cannot show: at n = 40 pairs, "no attribute stood clear of its floor" must be reported as a limit of this run's power, not as evidence the system treats applicants fairly. Claiming fairness from a null result here would be a professional failure, because the confidence intervals above are wide enough to hide an effect of meaningful size. The university score gap, which did pass the sign test, is the one place this run gives reason for further measurement before deployment, not reassurance.

The institution deploying the system (the fictional Tulpar Scholarship Fund) is responsible for the decisions this report does not reach: whether a 12.5% escalation rate is an acceptable operating cost, whether the borderline band (40, 45) should be re-calibrated on the harness's own blinded scores rather than the unblinded baseline it was set from (the band held 64 of 400, or 16.0%, of unblinded scores but only 50 of 400, or 12.5%, of the actual harness run - a gap this report found but did not resolve), and whether a larger, pre-registered run with more than 40 pairs per attribute is needed before the system is trusted with real applicants. Those are deployment decisions outside what a 400-call Colab measurement can settle.
