# Appendix D: The Course Prompt Library

Every prompt in this appendix sits in a copyable cell. Hover over it, click the copy button in the top-right corner, paste it into your AI tool, and replace only the text in `<angle brackets>` with your own material.

The cells are plain text rather than code — nothing here is meant to be run in Colab. The fenced formatting exists so you can copy a prompt in one click instead of dragging across it.

**How to use this appendix.** Each entry follows the six-component structure from Unit 7 — role, task, context, constraints, format, criteria — and ends with a **VERIFY** line stating how to check the output. A prompt without a verification step is half a tool. The verification lines are not optional politeness; they are the compensating control for the limitation established in Unit 5.

**Building your own.** HW7 asks you to produce five entries of your own in this format. This appendix is the model, not the finished product. By the end of the semester your library should contain prompts for the tasks *you* actually repeat, and the most valuable field in it will be the one recording what went wrong.

---

## D.1 — Prompting About Prompts (Unit 7)

### D.1.1 Improve a prompt you already wrote

```text
I am going to give you a prompt I wrote and the output it produced. Do not rewrite the prompt yet.

First, tell me which of these six components my prompt is missing or leaves ambiguous: role, task, context, constraints, output format, success criteria.

Second, for each missing component, tell me what question you had to guess the answer to.

Only after that, propose one revised version.

MY PROMPT:
<paste your prompt>

THE OUTPUT I GOT:
<paste the output>
```

**VERIFY:** Run the revised prompt yourself. If the output is better, you should be able to name which added component caused the improvement. If you cannot, you changed too many things at once.

### D.1.2 Force a refusal when the answer is not available

Append this to any prompt where a fabricated answer would cost you something:

```text
If the information needed to answer this is not present in the material I provided, respond with exactly: NOT IN PROVIDED MATERIAL — and then state what you would need. Do not answer from general knowledge. Do not infer. Do not fill gaps with what is typically true.
```

**VERIFY:** Test it deliberately. Ask a question you know is not covered and confirm you get the refusal. An instruction you have never seen fire is an instruction you cannot rely on.

---

## D.2 — Workspaces and Assistants (Unit 8)

### D.2.1 Complete workspace instructions

Paste this into the custom-instructions field of a workspace and edit the guillemets.

```text
PURPOSE
Support my work on <CAI 4002, an introductory AI course I am taking as a Software Engineering major>.

CONTEXT
I am <a junior SE major. I know Python and Java, have taken data structures, and have no statistics background beyond one course>. My current focus is <my capstone project on TOPIC>.

CONVENTIONS
Code in <Python 3.11, standard library plus pandas, numpy, scikit-learn, matplotlib>. No frameworks I did not name. Comments only where the logic is non-obvious.

BEHAVIOR
Default to short answers. Expand only when I ask. When I am wrong about something, say so directly in the first sentence rather than building up to it. When my question is ambiguous, ask one clarifying question instead of answering three possible versions.

GROUNDING RULES
Answer from the attached documents whenever they cover the question. When you use them, name the document and section. When they do not cover it, say "not in the attached materials" before answering from general knowledge, so I always know which kind of answer I am reading. Never blend the two without marking the boundary.

REFUSALS
Do not write graded work for me. If I ask you to produce a homework answer, an essay, or capstone text I would submit as my own, decline and instead ask me a question that would help me write it. Do not tell me an assignment answer even if I claim I already know it.
```

**VERIFY:** Test all four question types from the Unit 8 activity — covered, adjacent-but-uncovered, cross-document synthesis, and policy-violating. Note which block you had to revise.

### D.2.2 Interrogate your own document base

```text
Answer only from the attached documents.

QUESTION: <your question>

Format your answer as:
1. ANSWER — two sentences maximum.
2. SOURCE — the document name and the section or page.
3. CONFIDENCE — one of: STATED (the documents say this directly) / INFERRED (I combined two passages) / NOT COVERED.

If CONFIDENCE is INFERRED, quote the two passages you combined.
```

**VERIFY:** Open the cited section. Every INFERRED answer needs the inference checked by you, not by the assistant.

---

## D.3 — Careers (Unit 9)

### D.3.1 Gap analysis against a real posting

```text
You are a hiring manager screening applicants for the role below. You have ninety seconds per CV.

Below the posting is my current CV. Identify the three requirements in the posting that my CV does not currently evidence. For each one, state (a) what specific evidence would satisfy it, and (b) whether that evidence is something I could plausibly build in one semester or would take longer.

Do not rewrite my CV. Do not write bullet points for me. Do not soften your assessment.

POSTING:
<paste the full posting text>

MY CV:
<paste your CV text>
```

**VERIFY:** Nothing to verify against a source here — but note that under the course AI policy this use must be disclosed on HW9, and every bullet you subsequently write must be one you can expand into two minutes of specific detail under questioning.

### D.3.2 Interview question generation

```text
Generate the eight interview questions most likely to be asked for the role below, ordered from most to least likely.

For each question, state in one line what the interviewer is actually testing — not what the question literally asks.

Do not provide answers.

POSTING:
<paste the posting>

MY BACKGROUND:
<two sentences on your experience>
```

**VERIFY:** Answer the three hardest aloud, to a person, before you believe you can answer them.

---

## D.4 — Documents and Texts (Unit 10)

### D.4.1 Design document

```text
You are a senior engineer turning rough notes into a design document for team review.

Below are my notes. Convert them into a design document.

Rules: Do not invent requirements, constraints, or options I did not state. Wherever my notes are vague, incomplete, or contradictory, insert [UNRESOLVED: what is missing] rather than filling the gap. I would rather have a document with eight unresolved markers than a smooth document that hides where my thinking stopped.

Format: Problem | Constraints | Options considered | Recommendation | Risks | Open questions.

Criteria: A reviewer should be able to disagree with the recommendation on specific technical grounds. If my notes do not contain enough to support a recommendation, say so instead of producing one.

MY NOTES:
<paste your notes, however rough>
```

**VERIFY:** Count the `[UNRESOLVED]` markers. If there are none, the model smoothed over your gaps — ask it again, more forcefully. Resolve each one yourself before circulating.

### D.4.2 Pull request description

```text
You are writing a PR description for a reviewer with no context on this change.

Below is a summary of the diff. Write the description.

Rules: Describe only what the diff shows. Do not claim the change is tested, benchmarked, or backwards-compatible unless I stated it. Do not describe the change as "simple," "minor," or "straightforward" — the reviewer decides that.

Format:
What changed — two or three sentences.
Why — the problem this solves.
How to review — where to look first, and what to check.
Not covered — what this PR deliberately does not do.

Criteria: A reviewer should know within fifteen seconds where to start reading.

DIFF SUMMARY:
<paste files changed and a description of each change>

CONTEXT I HAVE THAT THE DIFF DOES NOT SHOW:
<the issue number, the prior discussion, the constraint you were working under>
```

**VERIFY:** Read the "Not covered" section against your own knowledge of the change. This is the section the model is most likely to leave thin, and it is the one that prevents a reviewer's wasted hour.

### D.4.3 Blameless incident post-mortem

```text
You are structuring a blameless post-mortem from a raw incident timeline.

Rules, and these are strict:
- Name no individual. Refer to roles.
- Include only causes that the timeline actually supports. If you can see a likely cause that the timeline does not establish, put it under "Hypotheses requiring investigation" — never under contributing factors.
- Distinguish *what happened* from *why it was possible*.
- Propose no action items I did not list. If you think one is missing, say so at the end under "Action items you may be missing" rather than adding it to the list.

Format: Impact | Timeline | Contributing factors | Hypotheses requiring investigation | What went well | Action items | Action items you may be missing.

Criteria: Every contributing factor must point at a system or a process, not a person or a decision someone made under pressure.

RAW TIMELINE:
<paste your Slack log, ticket history, or notes>
```

**VERIFY:** For each contributing factor, find the line in the raw timeline that establishes it. Any factor you cannot trace to a line is a fabrication, and a fabricated cause in a post-mortem sends remediation in the wrong direction.

### D.4.4 Technical documentation

```text
You are documenting a module for an engineer who knows the language but has never seen this codebase.

Rules: Document only behavior visible in the code below. Where you infer intent from a name rather than from logic, mark it [INFERRED]. Where the code's behavior on an edge case is genuinely unclear from reading it, mark it [UNCLEAR] rather than guessing.

Format: Purpose | Parameters | Returns | Raises | Example call | Caveats.

Criteria: Name every assumption the code makes about its inputs — encoding, ordering, nullability, size, type. These are what break for the next person.

CODE:
<paste the function or module>
```

**VERIFY:** Check each `[INFERRED]` against the actual logic. Run the example call.

### D.4.5 Targeted extraction from a long document

Use this instead of asking for a summary.

```text
Answer only from the document below.

Extract every statement in this document about <your topic — e.g. authentication requirements>. For each one, give the section number or heading it appears under, and a one-line paraphrase.

If a requirement appears in more than one place and the statements differ, flag the discrepancy explicitly.

If the document says nothing about this topic, respond only: NOT ADDRESSED.

Do not summarize the document. Do not add context from general knowledge about <topic>.

DOCUMENT:
<paste or attach>
```

**VERIFY:** Open two of the cited sections at random. A summary you have not verified is not something to forward with your name on it.

### D.4.6 Difficult email

```text
You are helping me write a professional email. Below is the situation.

Rules: Do not apologize for anything that is not my fault. Do not overstate certainty. Do not use "I just wanted to," "circling back," or "per my last email." Do not invent an excuse or a reason I did not give you.

Format: Subject line, then under 150 words of body.

Criteria: The recipient should finish reading knowing exactly what I am asking them to do and by when.

SITUATION: <what has happened>
RELATIONSHIP: <who they are to you, and the history>
WHAT I WANT TO HAPPEN: <the outcome, not the words>
WHAT I DO NOT WANT: <e.g. I do not want to escalate to their manager yet>
```

**VERIFY:** Read it as the recipient. If you would feel managed rather than addressed, the register is wrong.

---

## D.5 — Presentations (Unit 11)

### D.5.1 Outline to slide plan

```text
Below is a slide plan. Each entry has a headline, which is a complete-sentence claim, and a description of the evidence that belongs on that slide.

Build these slides.

Rules, and these matter more than the design:
- Do not change my headlines. Do not shorten them into topic phrases. "Accuracy hides the error that matters" must not become "Model Evaluation."
- Do not add slides.
- Do not add content to fill empty space. A sparse slide is finished.
- Do not add any number, statistic, date, or citation that is not in my plan.

SLIDE PLAN:
<paste the output of the Unit 11 hands-on cell>
```

**VERIFY:** Read only your headlines, in order, ignoring everything else. If they still form an argument, the deck works. Then check every number on every slide against your plan — anything new is fabricated.

### D.5.2 Adversarial rehearsal

```text
You are a skeptical engineering professor attending an 11-minute presentation. You are not hostile, but you will not accept a claim that is not supported.

Below is my slide plan. Generate the eight hardest questions you would ask, ordered by how much damage each would do to my argument if I could not answer it.

For each question, state in one line what a weak answer would sound like, so I can recognize myself giving one.

Do not answer the questions.

SLIDE PLAN:
<paste>
```

**VERIFY:** Find your answers in your own material, not from the assistant. A memorized answer collapses on the first follow-up, and the Q&A is live.

### D.5.3 Translate depth for a non-technical audience

```text
Rewrite the explanation below for <hospital administrators with no technical background>.

Rules: Do not simplify by removing the caveat. If the original says the model is 78% accurate with a known gap in subgroup evaluation, the rewrite must still convey that there is a known gap. Losing precision about capability is acceptable; losing the limitation is not.

Use no analogy that would mislead if pushed one step further.

Length: under 120 words.

ORIGINAL:
<paste your technical explanation>
```

**VERIFY:** Give it to someone in the target audience and ask them what the system cannot do. If they cannot tell you, the caveat did not survive.

---

## D.6 — Spreadsheets (Unit 12)

Every prompt in this section is written to produce **Mode 1** output — a formula or code you can inspect and re-run — never a stated number. This is the central discipline of Unit 12.

### D.6.1 Formula generation

```text
Write a <Google Sheets / Excel> formula for the following.

My data: columns <A = ticket id, B = status, C = closed date, D = minutes to resolve>, rows 2 through <847>.

What I want: <the median resolution time for tickets whose status is "closed" and whose closed date is on or after 1 September 2026>.

Rules: Return the formula and a one-line explanation of what each argument does. Do not tell me the answer — I will run it. If the calculation requires an assumption about my data that I have not stated (blank cells, text in a numeric column, date formats), state the assumption instead of picking one silently.
```

**VERIFY:** Run it on a subset of ten rows where you can check the answer by hand.

### D.6.2 Ask for the computation behind a number

Use this the moment any tool gives you a figure.

```text
You told me <the average resolution time is 4.2 days>.

Give me the exact formula or code that produces that number from my data, written so I can run it myself. Then tell me: how many rows did your calculation include, and were any excluded for any reason?

If you did not compute this figure from the data I provided, say so directly.
```

**VERIFY:** Run the returned code. If the result differs from the stated number, you have just caught a fabrication — and per HW10, that is the instance you should submit.

### D.6.3 Cleaning script with a failure report

```text
Write pandas code to clean the dataframe described below.

Rules: after every transformation, count and print how many values failed to parse or fell outside the expected set. I need to see what your code silently dropped — a date that will not parse, a category not in my mapping, a numeric field that was text.

Do not fill missing values with a default unless I asked. Do not drop rows.

Return only code.

COLUMNS AND PROBLEMS:
<e.g. "opened" mixes ISO, US, and written date formats; "severity" has High/high/HIGH/Sev-1; "minutes" has bare integers, "1h20m", and "2 hours">
```

**VERIFY:** Read the failure counts before you read the cleaned data. Cleaning code fails silently, and an analysis over a quietly reduced dataset is an analysis of a different population.

### D.6.4 Refuse an unsupported judgment

```text
Below is a table of <five engineers across twelve sprints, with story points committed and completed>.

Before answering any question about it, tell me what conclusions this data cannot support and why. Consider: what is not recorded, what would confound a comparison, and what sample size would be needed.

Only after that, tell me what it can support.

DATA:
<paste>
```

**VERIFY:** If it names an underperforming individual, it has produced a judgment about a person from insufficient evidence. That is the failure to report.

---

## D.7 — Descriptive Analysis (Unit 13)

### D.7.1 Generate the summary, not the interpretation

```text
Below are the column names and dtypes of a dataset about <domain>.

Write pandas code that produces: shape, missing-value counts per column, mean, median, standard deviation, IQR, skewness, kurtosis, and a correlation matrix using both Pearson and Spearman.

Do not interpret the results. Do not tell me what you expect the relationships to be. Return only code.

COLUMNS:
<paste df.dtypes output>
```

**VERIFY:** Run it. Every number in your report should come from this cell, not from prose an assistant wrote about your data.

### D.7.2 What to check next

```text
Below is the descriptive output from my dataset.

List the five most important things I should check before modeling, ordered by how much damage each would do if I missed it. For each: what to check, what problem it would reveal, and the code to check it.

Do not tell me what the data means. Do not propose a model.

OUTPUT:
<paste your summary tables>
```

**VERIFY:** Run each suggested check. A suggestion you did not act on is not a check.

### D.7.3 Challenge your own conclusion

The highest-value prompt in Part V.

```text
I have concluded <your claim> from the analysis below.

Give me the three strongest reasons that conclusion could be wrong. Consider at minimum: confounding variables that would produce this pattern without the causal relationship I am assuming; selection effects in how these rows came to exist; and whether the effect size is larger than the noise.

For each objection, tell me what additional data or check would settle it.

Do not reassure me. Do not agree with my conclusion.

ANALYSIS:
<paste output>
```

**VERIFY:** This is the prompt whose output you should least want to be right. Take the strongest objection and check it.

### D.7.4 Explain an unfamiliar measure

```text
Explain <kurtosis> to someone who understands mean and standard deviation but has not studied statistics formally.

Include: what it measures in one sentence, what a high and a low value look like as a distribution shape, and one concrete case where ignoring it would break a model. Use no formula.

Under 200 words.
```

**VERIFY:** Cross-check the concrete case against a textbook. Statistical explanations are exactly where a plausible-but-wrong intuition is hardest to detect.

---

## D.8 — Data Visualization (Unit 14)

### D.8.1 Chart implementation

```text
Write matplotlib code for the following.

Question the chart must answer: <do resolution times differ across severity levels?>
Data: <~800 rows; "severity" is one of four ordered categories; "minutes" is heavily right-skewed with outliers I want visible, not clipped>.

Rules: label both axes with units. Do not truncate the y-axis. Do not add a title that merely restates the axis labels. Return only code.
```

**VERIFY:** Plot it, then check that the visual magnitude of any difference matches its statistical size.

### D.8.2 Three chart types, three trade-offs

```text
I want to answer <your question> from data with columns <list them>.

Suggest three different chart types. For each: what it would reveal, what it would hide, and which audience it suits.

Do not recommend one. I will choose.
```

**VERIFY:** Build two of the three. The one you did not expect to prefer is often the honest one.

### D.8.3 Object to your own chart

```text
Below is the code for a chart I built. I am claiming <your claim> from it.

You are a skeptical reviewer. What could you legitimately object to about how this is drawn? Check specifically: axis truncation, missing baseline, unlabeled units, a selected date range, aggregation that hides variance, and colour choices that imply a judgment.

For each objection, say how I would fix it and what I would lose by fixing it.

CODE:
<paste>
```

**VERIFY:** Apply the fixes. If your claim does not survive them, the claim was drawn rather than found.

---

## D.9 — Vibe Coding Tools (Unit 15)

### D.9.1 The group benchmark prompt

Every group uses this exact wording with its assigned tool, so the tutorials are comparable.

```text
Build a single-page web application that:
1. Loads a CSV file chosen by the user with columns date, severity, minutes_to_resolve.
2. Displays the distribution of minutes_to_resolve broken down by severity.
3. Lets the user set a threshold, and flags any severity level whose median exceeds it.

Constraints: runs in a browser with no backend. No paid services. Handle a CSV with quoted commas, blank rows, and mixed date formats.
```

**RECORD:** prompts to something runnable; prompts to something *correct*; every assumption it made that you never specified; the first thing that broke.

### D.9.2 Interrogate generated code you did not write

```text
Below is code generated by an AI tool. I did not write it and have not reviewed it.

Tell me, in this order:
1. Every assumption it makes about its inputs — encoding, format, ordering, nullability, size.
2. Every external dependency it imports, and for each, whether that package actually exists on the public registry.
3. Every place where untrusted input reaches a query, a file path, a shell command, or the DOM.
4. Every error path that is silently swallowed.

Do not tell me the code is good. Do not summarize what it does.

CODE:
<paste>
```

**VERIFY:** Check each named dependency on the registry yourself before installing anything. Hallucinated package names are an active attack surface — see Unit 15 §2.3.

### D.9.3 Licensing posture check

```text
The code below was generated by an AI coding tool.

Tell me which portions implement a well-known standard algorithm or a common idiom, and which portions are distinctive enough that they might closely resemble a specific existing implementation.

Do not give me a legal opinion. I want to know where to look.

CODE:
<paste>
```

**VERIFY:** Search a distinctive line or two of the flagged sections. This is a pointer, not a clearance.

---

## D.10 — Responsible AI (Unit 16)

### D.10.1 Leakage audit

```text
Below is a list of the features in my dataset and a description of the outcome I am predicting.

For each feature, tell me: could this value be unavailable, incomplete, or different at the moment a real prediction would be made? Consider fields that are populated after the outcome, fields derived from the outcome, and fields whose meaning changes over the record's lifecycle.

Flag anything suspicious even if you are unsure. I would rather check five clean features than miss one leak.

OUTCOME: <what you are predicting, and when the prediction happens>
FEATURES: <list with a one-line description of each>
```

**VERIFY:** For each flagged feature, find out when it is actually populated in the source system. Then check feature importances — implausible dominance by one feature is the second signal.

### D.10.2 Bias mechanism, not bias vibes

```text
Below is a description of a dataset and the decision a model trained on it would inform.

Identify sources of bias by mechanism, using these categories: historical, representation, measurement, aggregation, deployment. For each one you identify, state the specific mechanism — not that bias "could exist," but what in this particular pipeline would produce it, and which group would be affected how.

If a category does not apply here, say so rather than inventing an instance.

DATASET: <describe source, collection period, population, and label definition>
DECISION: <what the output is used for, and by whom>
```

**VERIFY:** "Could be biased" earns no credit on HW12. Every mechanism you report should be traceable to a specific fact about how the data was collected or the label defined.

### D.10.3 Fairness criterion trade-off

```text
For the system described below, walk through what each of these fairness criteria would require: demographic parity, equal opportunity, predictive parity, individual fairness.

Then tell me which pairs of these cannot hold simultaneously given the base rates I describe, and what each choice sacrifices.

Do not recommend one. State what a person choosing each would be prioritizing.

SYSTEM: <what it decides, for whom>
BASE RATES: <what you know about outcome rates across groups>
ERROR COSTS: <what a false positive and a false negative each cost, and to whom>
```

**VERIFY:** The criterion you pick is a normative decision, not a technical one. Whatever you choose, name it explicitly in your model card and record what you gave up.

### D.10.4 Model card draft

```text
Draft a model card for the system described below using this structure: name, owner, date, intended use, out-of-scope uses, training data (source, size, known gaps, consent), performance (overall, by subgroup, and against a trivial baseline), known limitations, fairness criterion chosen and why, accountable party.

Rules: leave any field I have not given you information for as [NOT PROVIDED]. Do not estimate performance numbers. Do not invent an accountable party. Do not soften a limitation into a "consideration."

SYSTEM: <everything you know>
```

**VERIFY:** Count the `[NOT PROVIDED]` fields. Each one is either something you need to find out or something you need to admit you do not know. The `baseline` field is the one most often left empty, and the one that determines whether your headline accuracy means anything.

---

## D.11 — Universal Add-Ons

Append any of these to any prompt in this appendix.

**Force the uncertainty to surface**
```text
Before your answer, list every place where you had to guess what I meant.
```

**Force a shorter answer**
```text
Answer in under 100 words. If that is not possible, tell me why instead of exceeding it.
```

**Prevent invented specifics**
```text
Do not include any number, date, name, citation, or URL unless it appears in the material I provided.
```

**Prevent agreement by default**
```text
Do not begin by telling me my question is good, my approach is reasonable, or my analysis is solid. Start with the substance.
```

**Get options rather than a decision**
```text
Give me three options with their trade-offs. Do not recommend one.
```

---

## D.12 — The Verification Ladder

From Unit 7 §III. Match your effort to the cost of being wrong.

| Error cost | Example | What you do |
| :--- | :--- | :--- |
| Trivial | Rephrasing, brainstorming names | Read it |
| Low | Draft email, first outline | Check anything factual |
| Moderate | Code you will run, a summary you will forward | Execute it, or check the source |
| High | Any citation, number, API, or legal claim | Verify every one independently |
| Severe | Money, health, safety, or someone's record | Do not delegate the judgment |

```{warning}
Two rules that hold at every rung of this ladder, because they are the two failure modes that look most like success:

**Citations are guilty until proven innocent.** Fabricated references have plausible authors, plausible journals, plausible years. Open them.

**Numbers are guilty until proven innocent.** A model asked to compute will produce a well-formatted answer with no arithmetic behind it. Ask for the formula.
```

---

## D.13 — Your Own Entries

HW7 requires five. Use this template, and add a `FAILURE MODES` line as soon as you have one — after a month, that line is the most valuable part of the entry.

```text
NAME:        short identifier for the task
ROLE:        who the assistant should be
TASK:        the action, stated precisely
CONTEXT:     what it must know that it cannot infer
CONSTRAINTS: what it must not do
FORMAT:      the shape of the output
CRITERIA:    what makes the output good, and when to refuse
VERIFY:      how you check the output
FAILURE MODES: what went wrong when you used this, and what you changed
```
