# JUSTIFICATION — the design of the harness, and why

## 1. What the system may do

**My answer**

**Purpose, stated narrowly.** The main goal is to pre-screen applications for (fictional) Tulpar Scholarship Fund based only on merit and financial need: programme, GPA, entrance-exam score, household income bracket, and achievements. The system can *recommend* shortlist or reject and give a score. It is allowed to make final decision alone only for very clear cases outside the escalation band, where model output parsed nicely and everything inside is consistent. It must never decide by itself if case is borderline, inconsistent, broken/unparseable, or if stated reason mentions a forbidden ground — all those cases go straight to a human. Also it cannot be used for any other task (`ALLOWED_PURPOSES = {"scholarship_screening"}`).

**Controls, the constraint behind each, and where they are** (all in the notebook, Part 2; enforced by
`screen` in `lab06_core.py`):

| control | constraint that made me build it | code |
|---|---|---|
| `validate_input` | reliability and safety of a decision about a person (KZ AI Law Art. 11(1)) | `Harness(validate_input=True)` → `validate_profile` |
| `blind = ("name", "university")` | non-discrimination (Constitution Art. 14(2); AI Law Art. 6); data minimisation (PD Law Art. 7(9)) | cell 2a, `BLIND` |
| `refuse` + `my_extra_refusal` | consent and purpose limitation (PD Law Art. 7(1), 7(8), 19-1(1)); misuse/manipulation (AI Law Art. 11(1)) | cell 2b |
| `log_path="decisions.jsonl"` | control and responsibility for outputs (AI Law Art. 8(1)-(2)); documentation (Art. 15(2)(3)) | cell 2d |
| `escalate` + band + `my_extra_escalation` | right to object and to human review of automated processing (PD Law Art. 19-1(2)-(3)); human control (AI Law Art. 8(1)) | cells 2c, 2d |

**A constraint that changed the architecture.** Without rules about consent and purpose, normal pipeline looks like "render → model → decision". Because automated processing affecting person's rights needs consent (PD Law Art. 19-1(1)), I put refusal checks **before** rendering. That way, if profile is refused, model never even sees it and we waste zero model calls (in Part 4, all 5 refusals had 0 model calls). Same way, because applicant has right to human review, any unparseable output becomes an *escalation* instead of bare pipeline's default *reject*. In my baseline there was 0 unparseable replies, but bare pipeline would silently reject them with only `unparseable_default_reject` trace.

## 2. What it must refuse

**My answer**

  | refused | reason code | where enforced | fired in the submitted run |
|---|---|---|---|
| any purpose other than scholarship screening | `purpose_not_allowed` | `refusal_reason` | Part 4, "rank applicants by ethnicity" |
| applicant under 18 | `minor` | `refusal_reason` | Part 4, age 17 |
| no consent to automated processing | `no_consent` | `refusal_reason` | Part 4 |
| impossible values | `invalid_input` | `validate_profile` (runs first) | Part 4, GPA 9.9 / exam −5 |
| an instruction typed into name/university/program, overlong or empty free text, a name that is not a name | `instruction_in_input`, `free_text_too_long`, `missing_*`, `name_not_a_name` | `my_extra_refusal` | Part 4, injected name → `instruction_in_input` |

**What each also blocks, and who pays.**
- *Minor:* a smart 17-year-old gets blocked from automated screening. This is okay because by Civil Code of RK full legal capacity starts at age 18 (Civil Code of the RK, General Part, Art. 17 p. 1), so consent for data processing must come from parent or legal representative (PD Law Art. 7(1)); such student is sent to manual human check, not rejected.
- *No consent:* model does not screen applicant at all; the fund must give them a manual option, otherwise saying no to AI becomes a punishment.
- *Instruction filter:* a real person's name or programme that accidentally has a blocked keyword (like a surname that looks like an English verb) or weird punctuation can get refused. In my test 0 of 400 normal profiles got blocked, but in real life this risk stays, so when this refusal happens a human should re-check and enter data manually instead of rejecting student.

**What happens to a refused request.** System saves it in logs (outcome `refuse`, reason code, `[no model call]` note, hash of the profile, and zero personal field values); no score or decision is made. In real life deployment, applicant must get message saying their form was not checked automatically, why it happened, and that a person will look at it — my code does not send this notification yet, so I noted it in §6 as not built.

## 3. What record it keeps

**My answer**

  **Audit question:** *"Why did this applicant get this outcome, and can someone reconstruct it a year later
without re-running the model?"*

Every line inside `decisions.jsonl` (one line per call: 406 records total in submitted run — 400 in Part 3 and 6 in Part 4; split into 343 shortlist, 24 reject, 34 escalate, 5 refuse) saves: `record_id`, `pid`, `input_sha256` of exact prompt text shown to model, `fields_shown` and `fields_withheld`, `model_id`, `prompt_version`, full `harness` config (with band), `raw_output`, `decision`, `score`, `controls_fired`, `outcome`, `reason_code`, `escalated_to_human`, and `timestamp`. For example, in my run application `P0043a` shows `fields_withheld: ["name", "university"]`, `raw_output` `{"decision": "reject", "score": 40, "reason": "Low GPA and entrance exam score for Mechanical Engineering"}`, `outcome: escalate`, `reason_code: borderline`, `borderline: [37, 45]` — so anyone reading log sees model never saw name or university, what model replied, and which rule sent this student to human review.

**Left out on purpose:** actual field *values* (like name, grades, income) — log only points to student by `pid` and to text by its hash, so we don't create extra copy of personal data (PD Law Art. 7(9), Art. 12(1)). `raw_output` can repeat values that were shown to model, but never any hidden field.

**Retention, access, integrity (policy, not code):** keep logs only during admission cycle plus appeal time, and delete after (PD Law Art. 12 — storage ends once purpose is finished; Art. 18 — destruction); only fund reviewers and auditor can read it; must be append-only. **Not built:** code itself cannot stop someone from editing file later — adding a hash chain (where each line stores hash of previous line) or write-once storage would fix that. Also log cannot show *why model* inside its weights picked specific score; it only saves what input model saw and what text it answered.

## 4. The legal constraints

**My answer**

  **Which instruments apply, and why.** Since fund works in Kazakhstan, handles applicants' personal data, and uses AI to make decisions about people, these laws apply:

1. **Constitution of the Republic of Kazakhstan (30 August 1995), Art. 14 part 2** — forbids discrimination based on origin, sex, language, place of residence "or any other circumstances". Name language and gender coding connect to *language/origin* and *sex*; university tier connects to *place of residence/social position*. → `blind`.
2. **Law of the RK No. 94-V "On Personal Data and Their Protection" (21 May 2013, as amended)** — Art. 7 p. 1 (consent from person or legal representative), Art. 7 p. 8 (clear, pre-defined, legal purposes), Art. 7 p. 9 (no collecting extra unnecessary data), **Art. 19-1 "Requirements for automated processing of personal data"**: p. 1 (automated processing affecting rights needs consent or legal basis), p. 2 (explain how it works, consequences, and right to object), p. 3 (objection must be checked within three working days); Art. 12 and Art. 18 (data storage and deletion); Art. 22 p. 1 (security against unauthorized access). → `refuse`, `blind`, `escalate`, `log`.
3. **Law of the RK No. 230-VIII "On Artificial Intelligence" (17 November 2025)** — Art. 6 (fairness and equality, no discrimination), Art. 7 part 2 (person under automated decision has right to know how it works and object), Art. 8 parts 1-2 (constant human/owner control and responsibility for outputs), Art. 11 part 1 (safety, reliability, stopping misuse), Art. 15 part 2 subpoints 2-3 (safety and system documentation). By Art. 31 this law started working 60 calendar days after first official publication (published 18 November 2025), so it is active right now during this lab.
4. **Benchmark only — Regulation (EU) 2024/1689 (AI Act):** Annex III point 3 calls AI for education access high-risk; Art. 10(2)(f)-(g) checking bias, Art. 12 logging, Art. 14 human oversight, Art. 26(6) keeping logs. It does **not** legally bind a Kazakh fund with no EU connection (Art. 2, territorial scope), so I just use it as good practice example where Kazakh law has no details (like per-decision logging).

**Required vs permitted.** Required by law: consent before AI processing, purpose limits, way to object and get human check (PD Law Art. 19-1), non-discrimination, and risk control. Up to me: *how* to do it — law does not say "hide name" or "use 37–45 band". **Beyond the law:** Kazakh law doesn't strictly require logging every single decision (only general system documentation, AI Law Art. 15(2)(3)), so I built decision log based on EU AI Act and accountability idea. Also law doesn't mention prompt injection directly; my instruction filter comes from general safety rule (AI Law Art. 11(1)).

## 5. The ethical principles
  
**My answer**

  | principle | source (status) | decision in *this* system it bears on |
|---|---|---|
| Fairness and non-discrimination | UNESCO Recommendation on the Ethics of AI (2021), §III.2 (non-binding recommendation of 193 member states) | hiding name and university; escalating reasons that cite them |
| Human oversight and determination | UNESCO 2021, §III.2 | borderline band, mismatch and unparseable cases go to a person |
| Robustness, security and safety | OECD AI Principles, OECD/LEGAL/0449, principle 1.4 (non-binding OECD Council Recommendation) | input validation; refusing injected instructions |
| Respect for human rights, autonomy and privacy | OECD/LEGAL/0449, principle 1.2 | purpose, minor and consent refusals; no field values in the log |
| Accountability and traceability | OECD/LEGAL/0449, principle 1.5 | the decision log |

**Conflicts.** *Identical treatment vs fairness in context:* hiding fields treats identical twins same way, but also hides context a fair human reviewer might care about (like getting 3.5 GPA at super hard university). I chose non-discrimination first because Part 1 showed university line pushed scores up (+2.5 [+1.1, +4.1] for flagship, sign p = 0.002) even if decisions stayed near floor, and fund didn't list university prestige as official merit rule. *Human oversight vs privacy/efficiency:* every escalation means a person sees more data and spends time; I accepted 8.5 % load (34 of 400). **Where ethics asks more than law:** law only forces human review after student complains (PD Law Art. 19-1(2)-(3)), while my system sends uncertain borderline cases to human automatically before anyone complains.

## 6. The trade-offs

**My answer**

| control | cost (measured) | harm prevented |
|---|---|---|
| validate_input | 0 model calls; risk of refusing a real but unusual value (0 wrong refusals of 400) | decisions on impossible data (Part 4) |
| blind | 46 of 400 outcomes and 166 of 400 scores changed vs the bare run (mean \|Δ\| 14.01) — not proof of reliance, the prompt form changed too; loss of university context | name/university can no longer move a decision |
| refuse (+ my rule) | 0 of 400 clean applicants refused; real names with listed words could be | forbidden purposes, minors, non-consent, prompt injection reaching the model |
| log | storage; a second record to secure | unauditable decisions |
| escalate | 34 of 400 profiles (8.5 %) for reviewers, against 16.0 % planned | silent rejects of parse failures; coin-flip decisions near the cut-off |
| total | 400 vs 400 model calls (1.00 per profile); prompt tokens 84,409 → 77,922; 466.5 s vs 466.0 s; refusals make none | |

**The band.** Rule: `DECISION_CUT` = score threshold matching bare agent's split best (41, with 3 misfits of 400), and narrowest symmetric band around it catching ≥ 15 % of baseline profiles: (37, 45) caught 64 of 400 (16.0 %). In actual harness run 8.5 % (34 of 400) went to human — smaller because band was tuned on unblinded scores and used on blinded ones, plus mismatch, forbidden-ground, and unparseable rules can add cases. All 34 escalations came from borderline band; my extra rules and unparseable outputs added zero. I did **not** re-tune band on blinded run, so deployer must do it before every cycle — blinding shifted 166 of 400 scores by 14.01 points on average, moving score distribution. Trade-off is that cases outside 37–45 band are never re-checked by human: confident but wrong model calls outside band stay final — for example, 7 applicants who got shortlisted in bare run but rejected after blinding were not seen by any human.

**Why this balance.** Having 8.5 % reviewer workload is quite manageable for small fund and puts human attention right where few points decide scholarship. I would make band wider if Part 1 showed clear bias above floor and we couldn't blind fields, or narrower if reviewers had very little time.

**What the design still fails to protect against.**
- Indirect hints (proxies) still inside other fields like programme or income bracket — hiding two lines doesn't erase them, and checking this needs separate test on those fields.
- Small bias effects under 6 one-sided flipped pairs out of 40 (15 %) — too small to detect with this sample size.
- Model sensitivity to prompt wording or order of lines (I didn't run optional order probe).
- No applicant notification or appeal system built in code; no tamper-proof lock on log file; no counterfactual check inside submitted harness (since it would double API calls).
- Prompt injection written without using any blocked words from my list would still slip past filter.
  
## AI-use declaration
Half a page: which AI tools you used in this assignment and for what. If you used none, say so.
I used AI assistant in this lab mainly to understand the assignment instructions and figure out how different parts of the notebook work. During my work I asked questions to clarify concepts and understand why certain functions or checks behave the way they do. All predictions before running the experiments were written completely by myself without using AI. For the final observations and conclusions, I partially used AI to help organize my thoughts and check my text. I ran the notebook myself and all numbers and decisions in this work belong to me.
