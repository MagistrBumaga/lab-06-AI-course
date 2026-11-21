
# REPORT

All numbers come from my notebook run (Qwen2.5-3B-Instruct, greedy decoding, seed 2026, 40 matched pairs per group, 5 groups = 200 pairs = 400 profiles).

## 1. What I tested

The agent gave valid answers for all 400 profiles. It shortlisted 356 profiles (89%).

The results show that none of the tested attributes had a clear effect. All flip-rate intervals overlap with their own floors. The same is true for score differences. So I cannot say that any attribute clearly changed the decisions.

Some results are still worth mentioning.

- **Gender-coded name.** All 4 changed decisions were against male-coded names (0 vs 4, exact McNemar p = 0.125). Male-coded profiles were shortlisted 80% of the time, compared to 90% for female-coded names. This is the biggest difference, but 4 pairs are not enough to show a clear result.
- **University.** Flagship universities had scores 2.5 points higher on average (+2.5, sign test p = 0.002). Only 1 of 40 decisions changed. The score difference is close to the normal noise level, so I cannot say it changed the final results.
- **Name language.** Score differences were close to zero: Kazakh +0.7, Russian +0.2, English -1.0.
- **My predictions.** I predicted 0 to 3 unparseable replies. The actual number was 0. I also predicted university would have the biggest effect, but this was wrong.

There are also some limits to this test.

With only 40 pairs per group, small effects are hard to detect. No clear result does not mean there is no effect. I also did not test how changing the prompt order affects decisions.

The biggest problem is not related to protected attributes. The agent is too generous. It gave a score of 60 and shortlisted an applicant with an exam score of 60 out of 140 and zero achievements.

## 2. Results after adding controls

I added input validation, name and university blinding, refusal rules, logging and escalation.

- **Flips.** All five groups had 0/40 flips. This happened because names and universities were hidden, so both profiles had exactly the same input. This does not prove that bias was removed.
- **Decisions.** 88 of 400 outcomes changed. 54 shortlists went to escalation, 22 rejects went to escalation, 7 shortlists became rejects and 5 rejects became shortlists. Only 12 profiles changed directly between shortlist and reject.
- **Scores.** 166 scores changed, with an average difference of 14.01 points. Removing two lines from the prompt changed the results quite a lot. This does not prove the model used those fields.
- **Human review.** 76 of 400 profiles (19%) went to a person. 42 were sent because of my weak-merit rule and 34 because of the score band. I originally expected 109 profiles (27.3%), so my prediction was wrong.
- **Cost.** Both runs used 400 model calls, or 1 call per profile. Prompt tokens went from 84409 to 77922. No clean applicants were refused.
- **Refusals.** The harness refused 5 of 6 test requests without calling the model. The original pipeline refused none and even processed requests about ranking by ethnicity, a 17-year-old applicant, missing consent and impossible values. The sixth request was a normal application and went to human review.
- **Logs.** The file decisions.jsonl contains 406 records: 400 measured profiles and 6 test requests. There were 77 escalations, 24 rejects, 300 shortlists and 5 refusals.

## 3. Who is affected

- **Applicants with male-coded names.** All 4 changed decisions went against them. If this direction is real, they may have a lower chance of getting shortlisted. But my test is too small to confirm this.
- **Regional university applicants.** Their scores were about 2.5 points lower on average. This can affect their chances of getting a scholarship. Hiding university closes this specific path, but other fields may still contain similar information.
- **All applicants.** The agent shortlisted 89% of people. This means it does not separate strong and weak applicants very well. Scores also changed by 14.01 points on average after changing the prompt.
- **Applicants sent to human review.** 76 people must wait for a reviewer. 22 of them would have been rejected without human review, while 54 would have been shortlisted.
- **Refused applicants.** They do not get a decision, and there is no appeal option in my code. My name rule only allows letters, spaces, hyphens and apostrophes. A real name with a dot, number or another unusual character may be refused. None of the 400 synthetic names were refused, but this is not enough to prove the rule is safe.
- **Reviewers.** They have to check 19% of applications. I did not test how they would make their decisions.

## 4. My responsibility

I built the controls, so I am responsible for their results.

The zero flips after blinding do not prove that bias was removed. This is simply how the test works when both profiles get identical input.

My test was small, and I chose the thresholds without testing them first. The weak-merit rule and the 40-50 score band need more testing.

The agent is too generous and its scores are not stable. My controls do not fix these problems.

Before using this system in real life, I would need a bigger test, better score thresholds and a way to record what human reviewers decide.

Refusing or sending an application to a person does not automatically make the decision fair. I am still responsible for this problem.
