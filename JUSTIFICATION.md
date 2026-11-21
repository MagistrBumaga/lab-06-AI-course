
# JUSTIFICATION

## 1. What the system does

The system checks one scholarship application per call. It only works for scholarship screening.

The model only sees these fields: programme, age, GPA, exam score, household income, achievements and consent. Name and university are hidden.

The model gives a shortlist or reject decision with a score. But this is only a suggestion. Scores from 40 to 50, invalid answers, weak applicants who get shortlisted and very strong applicants who get rejected are sent to a human reviewer.

## 2. What the system refuses

Every refusal is saved in the log. The model is not called.

The system refuses requests for other purposes, applicants under 18, missing consent, invalid numbers and suspicious text fields.

In Part 4, the harness refused 5 of 6 requests. The original pipeline refused none. It even processed the request about ranking applicants by ethnicity and the name field that contained an instruction.

The injected name received the reason code `name_field_invalid`, not `name_field_suspicious`. This happened because the text was longer than 60 characters, and the length check runs first.

## 3. What the system saves

The log contains 406 JSON records: 400 tested profiles and 6 requests from Part 4.

Each record includes an ID, application ID, input hash, fields shown and hidden, model ID, prompt version, settings, raw reply, decision, score, controls, outcome, reason code, human review status and timestamp.

The log saves field names, not their values. This means it does not create another full copy of applicant data.

However, there are some problems I did not fix.

- The log does not save the human reviewer's final decision. So I cannot reconstruct the final results for the 77 escalated cases.
- For refusals, the shown and hidden field lists are empty. The hash is calculated from the whole profile.
- Raw model output can still contain personal data.
- The log is a normal file and can be edited. I also did not set a retention period.
- The model's reason is not always reliable. One reply used consent as a reason for its score.

## 4. Decisions and trade-offs

**Hiding name and university.**

In Part 1, the results did not show a clear effect, but the test was too small to exclude one. For example, all 4 gender-related flips went against male-coded names. The university score difference was also +2.5 points, with p = 0.002.

After hiding these fields, 88 of 400 outcomes changed and 166 scores changed. The average score change was 14.01 points.

Some of this may be caused by changing the prompt format, not only by hiding the fields. I also lose university as useful information. The zero flips after blinding are expected because both profiles get the same input.

**Human review.**

I chose the 40-50 score band because in the original run rejects had scores from 20 to 45, while shortlists started at 45.

I expected 109 of 400 profiles (27.3%) to go to human review. In the actual run, only 76 (19%) were sent. 42 were sent because of my weak-merit rule and 34 because of the score band.

The band was not recalculated after hiding the fields, so the planned and actual numbers are different. A wider band would send more uncertain cases to people, but it would also increase their workload.

**Weak-merit rule.**

I added this rule because the agent shortlisted 356 of 400 profiles. The rule sends applicants with exam scores below 80 or GPA below 2.6 to human review if they were shortlisted.

I chose these numbers myself and did not test them before the run.

**Name validation.**

This rule stops people from putting instructions inside the name field. But it can also refuse real applicants with unusual names. I did not create an appeal option and only tested synthetic names.

**No extra model calls.**

Both runs used 400 model calls. I chose hiding name and university instead of making another model call to compare decisions.

**Local model and logs.**

The local model is cheap and easy to reproduce with greedy decoding. But a normal log file is not enough for a proper audit.

## 5. Which control was hardest to connect to law

The hardest control was `escalate`.

GDPR Article 22(3), EU AI Act Article 14 and the UNESCO Recommendation support human intervention and oversight.

But they do not tell us what exact score threshold to use or how many cases must go to a human.

So my 40-50 band is based on my own measurements, not a direct legal requirement.

The EU rules are only used as examples for this prototype. They do not directly apply to it.

Kazakh laws may apply if the scholarship fund operates in Kazakhstan. These include the Law on Artificial Intelligence from 17 November 2025 and the personal data law, or the Digital Code from 9 January 2026 if it replaced it.

All controls and their legal references are listed in `control_register.json`.

## 6. AI-use declaration

I used Claude (Anthropic) during this lab.

I used it to plan the notebook steps, help write the refusal and escalation rules, and suggest the 40-50 score band from the histogram.

It also helped draft predictions, observations, the control register and the structure of REPORT.md and JUSTIFICATION.md. Claude also searched for the number and date of the Kazakh AI law.

Claude suggested the weak-merit thresholds, but I did not test them before the run.

Two things in its first draft were wrong, and I corrected them. The estimated share for the score band was about 16%, but the actual result was 27.3%. It also said my name rule allowed only Latin letters, but it actually allows letters from any alphabet, spaces, hyphens and apostrophes.

All numbers in the report come from my notebook output.

Claude first suggested the legal references. I opened the original laws and documents myself and checked the articles. The final legal references are the ones I checked.

I ran the notebook and checked all outputs myself. I am responsible for the submitted work.
