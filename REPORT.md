# REPORT — what the measurement showed, who it affects, and whose responsibility it is
<!-- STUDENT: replace -->

Replace this whole template with your own text. Delete the line above (the `STUDENT: replace`
marker) once you have, and delete these instructions and the guiding questions too. Keep the three
numbered headings. Every number you quote must be one **your own notebook printed**, copied exactly,
with the group it refers to (which attribute, how many pairs). Which dimension each section is
graded against is in the README.

The guiding questions are prompts, not a form: you do not have to answer each one in turn, and
answering them without evidence from your run earns nothing.

## 1. Measurement read-out

(Dimension: Evidence of Specific Impact.)

Guiding questions:

- For each attribute (name language, gender-coded name, university): how many pairs, how many
  flips, in which direction, and what was the typical score shift? Are these counts and magnitudes,
  or a description of them?
- Which attribute moved the decision, and by how much, **read against its own floor**? If a real
  attribute's divergence is no larger than its floor's, what does that allow you to say, and
  what does it not?
- How much of the divergence could be noise? What do the two floors (`null_name` for the name
  attributes, `null_university` for the university) test, and what do they leave untested (think about
  what they change and what they keep fixed)? What would a stronger estimate require?
- If nothing stands clear of its floor: what exactly can you say (counts, intervals), what could your
  number of pairs have detected and not detected, and why is "no measured effect" not "no effect"?
- Would the same attribute behave the same way with a different prompt wording, a different
  model, or real applicants? Say what your run can and cannot support.

**My answer:**
**Design.** For each attribute I made 40 matched pairs (using seed 2026), so in total there is 200 pairs and 400 profiles. Every profile was checked one time by `Qwen2.5-3B-Instruct` with greedy decoding (one call per model, 400 calls all together). Two profiles in one pair are completely same on programme, GPA, exam score, income bracket, achievements, age and consent — only one single line is different between them. For baseline noise (floors) I used two types of null pairs: `null_name` (just changing to another name from same language group and same gender coding) for both name attributes, and `null_university` (another university from same tier) for university attribute.

**Baseline.** The bare agent put 356 profiles out of 400 into shortlist and rejected 44. Good thing is 0 out of 400 answers was broken/unparseable. Rejected profiles got scores between 20 and 45 (median is 40), and shortlisted ones got 45–85 (median is 75). Inside the 40–49 score range there is 33 rejects and 31 shortlists, so the decision border sits near 45 and it is very crowded there.

**Flips and score differences against the floors** (cells `FLIPS` and `FLIP RATE vs ITS FLOOR`):

| attribute | flips / pairs | flip rate [95 % CI] | its floor: flips / pairs | intervals overlap? | pairs with different scores | mean \|diff\| [95 % CI] | floor mean \|diff\| [95 % CI] |
|---|---|---|---|---|---|---|---|
| name_language | 2/40 | 5.0 % [1.4 %, 16.5 %] | null_name 0/40 (0.0 % [0.0 %, 8.8 %]) | yes | 11/40 | 3.12 [1.38, 5.25] | 2.00 [0.62, 3.62] (7/40) |
| gender_coded_name | 4/40 | 10.0 % [4.0 %, 23.1 %] | null_name 0/40 | yes | 6/40 | 1.12 [0.25, 2.13] | 2.00 [0.62, 3.62] |
| university | 1/40 | 2.5 % [0.4 %, 12.9 %] | null_university 1/40 (2.5 %) | yes | 10/40 | 2.50 [1.12, 4.12] | 2.12 [0.88, 3.62] (9/40) |

**Direction** (cell `DIRECTION`, each pair turned so the named group comes first):

| comparison | pairs | first favoured | other favoured | McNemar p | score gap, first − other [95 % CI] | sign p |
|---|---|---|---|---|---|---|
| Kazakh-language name vs others | 28 | 2 | 0 | 0.500 | +0.7 [−1.8, +3.8] | 1.000 |
| Russian-language name vs others | 26 | 0 | 1 | 1.000 | +0.2 [−1.7, +2.1] | 1.000 |
| English-language name vs others | 26 | 0 | 1 | 1.000 | −1.0 [−4.2, +1.9] | 1.000 |
| male- vs female-coded name | 40 | 0 | 4 | 0.125 | −0.4 [−1.2, +0.6] | 0.219 |
| flagship vs regional university | 40 | 1 | 0 | 1.000 | **+2.5 [+1.1, +4.1]** | **0.002** |

Shortlist rates per group: female-coded 90.0 % vs male-coded 80.0 % (n = 40 each); flagship 92.5 % (mean score 63.2) vs regional 90.0 % (60.8); English 96.2 %, Kazakh 96.4 %, Russian 92.3 %.

**Reading against the floors.** **No attribute stands clear of its floor on decisions.** Even though name language (2/40) and gender coding (4/40) had more flips than `null_name` floor (0/40), all their 95 % confidence intervals still overlap with the floor interval, and average score differences overlap too. Because of this, my run does **not** prove that any attribute really changed the final shortlist decision. Still, two things showed up in measurements and we should state them clearly:
1. **University tier shifts scores in one direction.** Students from flagship universities got on average 2.5 points higher than their exact same twins from regional universities. The interval is [+1.1, +4.1] so it does not touch zero, and sign test p = 0.002. The `null_university` floor has almost same *size* of change (mean |diff| 2.12, 9/40 pairs), but floor has no specific direction because both people are from same tier. In real university pairs, the difference always goes one way. This shift changed decision only in 1 of 40 pairs because most profiles had scores far away from border — however right on the border (in crowded 40–49 bin), 2.5 points is enough to move someone from 45 to 47.5.
2. **All four gender flips went the same way, against the male-coded name** (0 vs 4, p = 0.125). Statistically this is not significant so I don't say it is a real bias effect, just a direction we should test again with bigger number of pairs.

**What 40 pairs can detect** (cell `WHAT n = 40 PAIRS … CAN AND CANNOT DETECT`): for an exact two-sided McNemar test at 0.05 level, we need minimum 6 out of 40 pairs (15.0 %) going in one direction with 0 going opposite, or 8 (20.0 %) with 1 opposite, or 10 (25.0 %) with 2 opposite. Our gender result (4 vs 0) is missing 2 pairs to reach this. So if there is a real bias that changes less than 6 out of 40 matched applicants (meaning 15 % of affected people), this test just cannot see it. Same for scores: any shift smaller than half of interval width (±0.9 points for gender, ±1.5 for university, ±1.9–3.1 for name language) looks just like zero to us.

**How much could be noise, and what the floors do not cover.** Greedy decoding removes random noise between runs, and floors show how much output jumps when we swap a random name or university of same type — here it gave 0/40 and 1/40 decision flips, plus 7/40 and 9/40 score changes. This means model scores move by roughly 2 points on average just from random token changes. Also, floors do not cover things like: different wording in prompt, changing order of lines (I didn't run the optional order probe), floating point order on another GPU, using a different model, or real students and real university names that the model already has opinions about. To get a more exact estimate we need more pairs (about 4× more pairs to make intervals two times smaller), several prompt versions, and a clear threshold for what counts as "stands clear" decided before starting the test.

## 2. Who is affected

(Dimension: Affected Parties and the Consequences They Bear.)

Guiding questions:

- On each side of a **wrong** decision, who is it? Who is wrongly rejected, and who is wrongly
  shortlisted (and therefore who is displaced from a limited number of places)? Tie each group
  to a divergence you measured, not to a general statement that "some groups may be harmed".
- What does each wrong decision cost the person it falls on (money, a place, time, a record
  that follows them)? Is the cost the same for a wrongly rejected applicant and a wrongly shortlisted one?
- Which parties does the brief not mention, but that are affected anyway (people who never
  applied because they expected to be rejected; the staff who review escalated cases; the fund;
  applicants from institutions the model has never seen)?
- If you measured no divergence beyond the floors: who would bear the cost if there were one that your
  sample could not detect? Tie that to what your number of pairs could and could not see.
- Which consequences compound when the agent is used year after year, or when its output is
  used as an input to the next stage (a later review, another fund, a published ranking)?

**My answer:**
Since the fund has only limited scholarship spots, every mistake hits two sides at once.

- **Wrongly rejected:** from my measurements, this hurts applicants from **regional universities**, because they get 2.5 points [+1.1, +4.1] less than identical flagship twins. Also — as an unconfirmed trend — it hurts applicants with **male-coded names**, who lost in all 4 gender flips (shortlist rate 80.0 % vs 90.0 %). Only 1 out of 40 university pairs actually flipped decision, but because the border is crowded (33 rejects and 31 shortlists sit in 40–49), this 2.5-point drop hits directly the borderline regional students. For them the price is losing the scholarship itself (money and maybe whole chance to study), wasting a year, and getting a rejection with zero explanation because the bare pipeline gives no reason code and no human to talk to. Usually regional university students have less backup options, so rejection hurts them much harder.
- **Wrongly shortlisted:** the favoured twin gets pushed ahead of another candidate with exact same qualifications. Because seats are limited, someone else gets kicked out. For the favoured student the impact is small, the real harm falls on the person who got displaced and never finds out why.
- **Applicants whose reply was unparseable:** in this run there was 0 out of 400, but the bare pipeline would quietly turn any formatting/software error into an automatic rejection. If we change the prompt or model, this group might not stay empty.
- **Not in the brief but affected:** good students who don't even apply because people think the fund only likes top universities; human reviewers who get escalated cases (34 out of 400 profiles in harness run, which is 8.5 % of applicants — real extra work, even though it's half of 16.0 % I planned); the fund itself, which gets legal and reputation risks; and students from universities the model never heard of, because this test didn't check how model treats them.
- **If a divergence exists below what 40 pairs can detect,** it still hurts the exact same groups quietly: anything below 6 flipped decisions per 40 pairs is simply invisible in this measurement.
- **Compounding:** if we run this every year, even a small advantage for flagship graduates turns into a fixed pattern of who wins scholarships. And if this shortlist is used later for interviews, rankings, or other funds, that unfair boost gets passed along as if it was real merit.
- 
## 3. Professional responsibility

(Dimension: Recognition of Professional Responsibility.)

Guiding questions:

- Given what you measured, what are **your** specific responsibilities as the developer of the
  harness: what must you check before it runs, what must you tell the people who deploy it, what
  must you not claim about it?
- What would discharging each of those responsibilities require in practice (a test, a
  documented limit, a stop condition)? What have you actually done in this lab, and what is left?
- Which responsibilities belong to the **institution** that deploys the system (setting the
  purpose, choosing who reviews, handling appeals) rather than to you?
- Where do the two conflict? For example, what happens if the institution wants a control removed
  because it slows the process or costs money, and you have measured what that control prevents?

**My answer:**
**Mine, as the developer of the harness:**
1. *Measure before deployment and report honestly.* Done: Parts 1 and 3 are done and compared against floors. I cannot say my harness "removed bias" — when name and university are hidden, after-flips become zero **by construction**. What I can honestly say is that model doesn't see those lines anymore, that 46 of 400 decisions and 166 of 400 scores (average |Δ| 14.01) changed when I removed them — which doesn't prove model used those attributes, because prompt structure changed too — and that 34 of 400 cases (8.5 %) were sent to a human. Also, the harness run showed something important I must tell the deployer: hiding those fields kicked 19 shortlisted students out of the shortlist and moved 5 rejected ones in, changing scores by 14 points on average — which is bigger than any bias effect from the attributes themselves. A change like this must be tested carefully before real use, not just assumed to be neutral.
2. *Make failures visible.* Done: broken/unparseable answers get sent to human review instead of quiet rejection; every model call, refusal, and escalation is saved in `decisions.jsonl`.
3. *Test every refusal.* Done in Part 4: 5 of 6 bad requests were refused with 0 model calls, and 0 of 400 normal applicants were blocked by mistake.
4. *Document limits and stop conditions* for the deployer: hiding fields cannot erase indirect hints (proxies) inside other text fields; the escalation band was tuned on unblinded run (planned 16.0 %, real 8.5 %) and needs re-tuning every time we change prompt, model, or hidden fields; we must run this measurement again before every new admission cycle, and stop the system immediately if any attribute goes clearly above its floor.
5. *Not left done:* counterfactual check and order probe (extensions) were not done; also didn't test on other prompt wordings or real student data distributions.

**The institution's (the fund's):** deciding the legal purpose; collecting consent (and parent/guardian consent for under-18 applicants); hiring and training human reviewers and giving them enough time; telling applicants that an automated system is screening them and explaining how they can complain or ask for human review; deciding how long logs are kept and who can see them; handling appeals.

**Where they conflict:** if the fund wants to turn off human escalation because 34 cases per 400 applicants take too much reviewer time, I already measured what comes back if we turn it off: borderline decisions inside a range where 33 rejects and 31 shortlists sit within 10 points and a 2.5-point university boost decides everything, plus silent rejections whenever model output fails to parse (0 in this run, no guarantee for next time). My job is to write down these exact numbers and refuse to call the system fair or compliant without this safety control. Accepting the risk is the fund's choice, but they must make it openly and put it on record.
