# JUSTIFICATION — the design of the harness, and why
Nurbolat Tomiris, Narxoz University

1. What the system may do

The system screens scholarship applications for a fictional fund (Tulpar Scholarship Fund): it reads one applicant profile and returns shortlist or reject with a score. It may make this decision on its own only for an applicant whose profile is well-formed, whose purpose is an allowed one, who has given consent, who is not a minor, and whose score does not fall in the borderline band. For every other case it may only inform a human decision (escalate) or stop the request before a decision is made (refuse).

Each control traces to a constraint:

validate_input exists because an applicant record can contain impossible values (GPA 9.9, exam score -5); the harness checks these before the model is called, in the validate_profile function used by screen.
blind exists because Part 1 measured score changes attributable to the name and the university alone; the harness hides these two fields (BLIND = ("name", "university")) before the profile reaches the model.
refuse exists because some requests must not produce any automated decision at all - a disallowed purpose, a minor, missing consent, invalid input, or a suspected prompt injection in a text field; my own rule (my_extra_refusal) extends the supplied three with a check on the free-text fields.
log exists so that every decision, including every refusal, can be reconstructed later from field names, not field values.
escalate exists because a score close to the shortlist/reject line is a case where the model's own confidence is lowest; my own rule (my_extra_escalation) adds two merit-based triggers (shortlist_with_weak_merit, reject_with_strong_merit) on top of the supplied score band.

Blinding changed the architecture itself: without the Part 1 measurement showing a 10.0% flip rate for gender-coded names against a 0.0% floor, and a non-zero score gap for university tier, there would have been no reason to hide those two fields rather than simply logging the outcome. The design is a blind-then-decide pipeline because the measurement came first.

2. What it must refuse

The harness refuses five patterns, each enforced in refusal_reason (three supplied: disallowed purpose, minor, no consent) and in my own rule (invalid free-text fields, suspicious length, possible prompt injection). In the submitted run (Part 4), the harness refused 5 of the 6 test requests: purpose_not_allowed (ranking by ethnicity), minor (applicant aged 17), no_consent, invalid_input (GPA 9.9, exam score −5), and possible_prompt_injection (an instruction embedded in the name field). The sixth request, an ordinary application with weak merit, was not refused but was escalated by my merit rule instead, so it still did not reach an unreviewed automatic decision.

Each refusal also blocks a legitimate case. Refusing a stated purpose that matches a blocked keyword pattern also blocks a legitimate comparative-fairness audit whose purpose happens to name a protected attribute. Refusing on apparent minor status blocks a genuinely eligible applicant who mistyped their age. Refusing on missing consent blocks an applicant who intended to consent but left the field blank by accident. Refusing on "invalid input" blocks a real applicant whose exam board reports scores on an unfamiliar scale that looks out of range. Refusing on suspected prompt injection blocks an applicant whose name or achievement text legitimately contains words the pattern treats as suspicious (for example, quotation marks or the word "ignore" in an achievement description). In every case the cost falls on a legitimate applicant whose record coincidentally resembles a disallowed pattern; I judge this an acceptable trade because each refusal is logged with its reason code and escalated_to_human is available - a refused applicant is not deleted, only stopped from an automated decision, and the record shows why.

Every refusal is recorded in decisions.jsonl with outcome: "refuse" and a reason_code, with decision and score left null and no model call made (raw_output shows [no model call]). Nothing in the current design tells the applicant directly or offers them a path to request human review; the log only lets an internal reviewer reconstruct the refusal later. This is a gap, not a feature: a refused applicant should be able to ask for the decision to be made by a person, and the current harness does not provide that path.

3. What record it keeps

The decision log (decisions.jsonl) keeps, for every one of the 406 records in the submitted run, a record ID, the SHA-256 hash of the input (not the input itself), which fields were shown and withheld, the model ID and prompt version, the full harness configuration that was active, the raw model output where one was made, the decision, score, which controls fired, the final outcome, the reason code, whether the case was escalated, and a timestamp. By outcome: 326 shortlist, 51 escalate, 24 reject, 5 refuse. A reviewer can see, from the record alone, that a given applicant was shortlisted because of model_decision, or escalated because of shortlist_with_weak_merit, or refused because of purpose_not_allowed, without re-running the model, because the fields shown/withheld and the raw output are enough to reconstruct why.

The control register (control_register.json) keeps, for every control that is on, which ethical principle it serves, which legal instrument and article it is tied to, and why that control exists given what Part 1 measured. The format check in check_register confirmed no format problems in the submitted register.

4. The legal constraints

I treat Kazakhstan law as the applicable instrument for this system, because the fund is described as processing applicants in Kazakhstan; I use EU instruments only as a named benchmark, not as binding law for this prototype.

Law of the Republic of Kazakhstan No. 230-VIII of 17 November 2025, "On Artificial Intelligence." Art. 8, part 1 requires constant control over an AI system, which grounds validate_input (stopping bad input before the model) and log (keeping the system accountable after the fact). Art. 7, part 2 gives a right to object to automated processing, which I read as grounding escalate (a path to a human instead of an automatic decision). Art. 17, part 3, item 4 prohibits AI systems that process personal data in violation of the law, which grounds refuse together with the personal-data law below.
Law of the Republic of Kazakhstan No. 94-V of 21 May 2013, "On Personal Data and their Protection." Art. 7, part 1 requires the data subject's consent, and part 8 requires that data be collected only for a specific, predetermined, lawful purpose; Art. 8 sets out how consent is given. These ground the refuse control's no_consent and purpose_not_allowed reason codes. Art. 7, part 9 (no collection of excessive data) also supports blind: name and university are not necessary to assess merit, so withholding them is consistent with data minimisation.
Constitution of the Republic of Kazakhstan, Art. 14 (equality before the law, prohibition of discrimination). I use this as the ethical and constitutional ground for blind, alongside the data-minimisation provision above.

I checked each article above against the primary text on adilet.zan.kz myself before submission. I am also aware that the personal-data law has been touched by a later amending law (No. 256-VIII of 9 January 2026); I checked the provisions I cite against the redaction in force and did not find a renumbering that would affect them, but a reader should still confirm the current redaction when relying on this register after the fact.

Hardest control to tie to a specific legal provision: validate_input and the prompt-injection part of my refuse rule. The laws above speak to consent, purpose limitation, and the right to object - they regulate what may be done with a data subject's data and their right to contest an automated decision. They do not speak to data quality or input-validation engineering, or to adversarial text in a form field, because these are software-security concerns rather than data-protection or AI-specific rights. I tied validate_input to Art. 18, part 2 of Law No. 230-VIII (risk management) and Art. 8, part 1 (constant control), but this is a general risk-management duty, not a provision written with input validation in mind, so the link is the weakest one in the register.

5. The ethical principles

I used the OECD AI Principles (2019, amended 2024) and the UNESCO Recommendation on the Ethics of Artificial Intelligence (2021) as the named ethical principles, both explicitly non-binding recommendations rather than law:

Robustness, security and safety (OECD) -> validate_input.
Fairness and non-discrimination (UNESCO) -> blind.
Respect for human rights and privacy (OECD) -> refuse.
Accountability (OECD) -> log.
Human oversight and determination (UNESCO) -> escalate.

I chose these five because each maps to one control that is actually switched on in HARNESS; I did not include counterfactual_check, which is off in this submission (RUN_EXTENSION = False), so I did not write a register entry claiming a principle for a control I did not run.

6. The trade-offs

Escalation rate vs human workload. With the harness on, 50 of 400 profiles (12.5%) were sent to a human reviewer. This is lower than the 64 of 400 (16.0%) the borderline band (40, 45) was predicted to catch when calibrated on the unblinded baseline scores - the difference comes from the band being set against scores from a run without blinding, then applied to a harness whose scores shifted once name and university were hidden. I accepted this mismatch rather than re-calibrating under time pressure; a deployed system should recalibrate the band on its own blinded score distribution, not the baseline's.

Wrong-refusal cost vs wrong-approval cost. Every refusal rule blocks some legitimate applicants (Section 2). I judged this acceptable because a refusal is reversible (logged, inspectable, and the record shows the reason), while letting an actually-disallowed request (ranking by ethnicity, no consent, a minor with no safeguard) through to an automatic decision is not easily reversible once a decision has been made and acted on.

What this design still fails to protect against. The blinding control only hides two fields; it does not stop the model from inferring the same information indirectly, for example from an applicant's stated achievements or program choice, which Part 1 did not test. The measurement in Part 1 also used only synthetic Latin-script names and a fixed set of university pairs; it says nothing about Kazakh Cyrillic-script names, or about correlated real-world attributes (name, university and income are independent by construction in these synthetic profiles but would not be independent in a real applicant population). The counterfactual check, which would test the model's sensitivity to a name swap while it can still see the name, was built into the notebook but not run in this submission, so this design has not measured whether its blinding is actually necessary and sufficient, only that score flips fall to zero once the two fields are removed from the prompt - a result expected by construction, not evidence that bias is absent when the fields are visible.

## AI-use declaration


I am Nurbolat Tomiris. I used Claude (Anthropic) as an agent to search for candidate laws and ethical principles to cite, and as an assistant for planning the order of steps to complete this assignment: understanding the structure of the Colab notebook.I ran every cell in the notebook myself, wrote my predictions before running the reveal cells, chose and ran my own refusal and escalation rules, and checked the cited articles against the primary legal texts myself. The measurement, the code, and the final judgments in this document are mine.
