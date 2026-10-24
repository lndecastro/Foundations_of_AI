# Unit 12: Spreadsheets

```
Session 18 | Oct 19 | HW10 (Office Portfolio) due Oct 21
```

Spreadsheets are where AI assistance is most useful and most dangerous, and the two facts have the same cause: **a spreadsheet looks authoritative.** A number in a cell carries an air of computation whether or not any computation occurred.

This unit is built around one in-class activity, **The Audit at Gulf Coast Clinics**, an escape room in which teams open five locked doors with numbers computed from a messy clinic visit log. The concepts below explain what the doors were testing, the hands-on cells let you rerun each trap on new data, and the debrief compares your method with what an AI model does when it is handed the whole file.

This session closes Part IV and sets up Part V. The habits you build here, such as never trusting a number you did not verify and always separating computation from generation, are the habits that make Units 13 and 14 safe.

## Learning Objectives

After completing this unit, you will be able to:

- Distinguish AI that **writes a formula** from AI that **states a number**, and prefer the first.
- Describe a dataset precisely enough for an AI tool to write correct formulas for it.
- Generate, run, and audit formulas for cleaning, counting, and summarizing data.
- Recognize when a correct-looking number answers the wrong question, and when data cannot support a decision about a person.
- Decide what data may and may not be shared with an AI tool.

## Part I — The Central Distinction

There are two fundamentally different things an AI tool can do with your data, and confusing them is the source of nearly every serious error in this domain.

**Mode 1 — It writes code or a formula that computes the answer.**
You get `=SUMIFS(D:D, B:B, "closed", C:C, ">="&DATE(2026,9,1))` or a few lines of pandas. The computation happens in your spreadsheet or your runtime, deterministically. The AI is generating a *specification for arithmetic*, and if the specification is wrong you can read it and see that it is wrong.

**Mode 2 — It reads your data and tells you the answer.**
You get "the average resolution time was 4.2 days." This number was **generated as text**, by a system whose objective is producing likely continuations. Unit 5's argument applies with full force: it may be right, it may be approximately right, and it may be entirely invented, and it will look identical in all three cases.

```{warning}
**Prefer Mode 1 always.** When the answer arrives as a formula you can inspect and re-run, you have an auditable artifact. When it arrives as a sentence, you have a claim.

If a tool gives you a number, ask it for the formula that produced the number, and if it cannot produce one, you have learned something important about where the number came from.
```

## Part II — Where AI Helps, and What It Cannot Do for You

AI tools are genuinely good at the mechanical parts of spreadsheet work:

- **Formula generation.** Describing what you want in words and receiving a formula is far faster than remembering `INDEX/MATCH` argument order, and it is easy to verify: run it on rows where you know the answer.
- **Cleaning logic.** Inconsistent dates, mixed units, stray whitespace, names in three formats: AI-generated cleaning formulas and code handle these well, and the results are checkable by inspection.
- **Formula debugging.** A `#REF!` or `#VALUE!` in a nested formula is exactly the kind of syntactic puzzle these tools solve.
- **Aggregation design.** Describing the summary you want and getting the grouping logic for a pivot table.

What the tool cannot do is **see what you did not tell it.** When you describe your data, the description is the whole world the model works in. If you do not notice that some dates are stored as text, you will not mention it, and the model will write a clean, plausible formula that silently skips those rows. Call this the **description gap**: every error the AI cannot catch sits between what the data contains and what you said it contains.

The escape room was designed around that gap.

## ⚙️ In-Class Activity: The Audit at Gulf Coast Clinics

Teams of 4 to 5, 45 minutes, one workbook, one online form.

**The story.** It is audit day at Gulf Coast Clinics, which runs two walk-in clinics in Lee County, Florida. The state auditor arrives in 45 minutes. The visit log was merged from two front-desk systems and never cleaned. Five locked doors stand between your team and the exit, and each opens only for a number computed from the full log.

**The AI rule.** The visit log is patient data, so **the AI never sees it.** You look at the data, the AI writes the formulas, and Google Sheets does the computing.

| Allowed | Not allowed |
| --- | --- |
| Describing the columns and what you see in them | Uploading the workbook to any AI tool |
| Typing made-up example rows | Pasting, typing, or screenshotting real rows |
| Pasting formulas and error messages | Asking the AI to state a number for you |
| Pasting summary tables you computed (counts or rates by group) | Copying another team's codes or formulas |

In a real clinic, patient records cannot be pasted into a public chatbot. In many jobs you will be allowed to upload data to an approved AI tool; the debrief and Unit 13 use that workflow. The rule here is not a handicap: it is what makes your contribution visible.

**The doors.**

| Door | Room | Task |
| --- | --- | --- |
| 1 | Intake | Count the valid visits |
| 2 | The Screenshot | Find the average wait across all valid visits |
| 3 | The Deadline | Count Urgent visits that broke a new 60-minute rule |
| 4 | Which Clinic? | Decide which clinic should be the model for the other |
| 5 | The Last Door | Answer the board's request to put a doctor on probation |

The rooms are chained: each uses the result of the one before. A wrong value shows a nudge; a correct one opens the door and reveals a stamp word.

**The Team Log** is the first sheet of the workbook and the basis of your grade. For every door, it records the stamp word, the formula you ran, the prompt you gave the AI, and every wrong value you tried with one line on why it was wrong.

| Component | Points |
| --- | --- |
| Doors opened (stamp word logged) | 5 × 1 |
| Formula and prompt logged, per open door | 5 × 0.5 |
| Room 4 explanation, with numbers | 1 |
| Room 5 justification, with numbers | 1 |
| Wrong-values list | 0.5 |
| **Total** | **10** |

Breaking the AI rule costs 5 points. The first team out earns 1 bonus point and the second 0.5.

## Part III — What the Five Doors Teach

Every door was built around one trap, and every trap survives correct arithmetic. None of them is caught by checking the math; each is caught by checking **what was computed, over which rows, for which question.**

**Door 1 — Cleaning fails silently.** The log held duplicate rows, blank dates, "TBD" dates, and wait times typed as text, such as `25 min`. A formula that treats text as missing drops real visits without an error message. The habit: before trusting any count, count what did not survive each cleaning step.

**Door 2 — Correct arithmetic, wrong population.** The clue offered a figure a chatbot computed from the first rows of the log. Its arithmetic was fine; it simply answered for an excerpt while sounding like it answered for the whole file. A second trap hid underneath: spreadsheet `AVERAGE` skips cells that contain text, so even a full-column average was computed over fewer visits than it appeared to be. The habit: for every number, ask how many rows went into it.

**Door 3 — Storage and boundaries.** Some dates were real dates and some were text that only looked like dates, so a date comparison quietly skipped them. The rule said "more than 60 minutes," and `>` and `>=` give different counts when patients wait exactly 60. "Urgent" was typed several ways. The habit: click into the cells, and read the rule's boundary word by word.

**Door 4 — Aggregation hides case mix.** The clinic with the higher overall resolution rate was worse at both Urgent and Routine visits. It looked better only because most of its visits were easy ones. This is **Simpson's paradox**: a comparison of totals reflects the mix of cases as much as the quality of the work. An AI asked "which clinic is better?" usually answers from the totals. The habit: compare like with like before you compare overall.

**Door 5 — Judgments about people.** The doctor with the lowest resolution rate had seen very few patients, nearly all of them the hardest cases. Two separate problems, a small sample and a hard caseload, made the ranking meaningless as evidence about the doctor. The only way out was to refuse to name anyone and to justify the refusal with numbers. The habit: a number about a person needs more scrutiny than a number about a process, because someone may act on it.

```{note}
The common thread is the description gap. A team that never noticed the text dates never mentioned them, and the AI's formula skipped them. A team that asked the AI "which clinic is better?" got an answer to the question it asked, not the one the board needed. The AI did its part every time; the noticing was the human part.
```

## ⚙️ Hands-On: How Cleaning Fails Silently

Doors 1 to 3 used a clinic log. This cell rebuilds the same traps in a different log, a city's 311 service requests merged from a phone line and a web form, and asks one question: how many pothole requests received on or after September 1 took more than 7 days to close? Three settings at the top switch each trap on or off.

```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta

# ---- The settings you change ---------------------------------------
PARSE_TEXT = True    # convert dates and day counts typed as text
DEDUPE     = True    # remove rows that are exact copies
STRICT     = True    # True: "more than 7 days" (>), False: ">= 7"
# ---------------------------------------------------------------------

# A city 311 service-request log, merged from a phone line and a web form.
rng = np.random.RandomState(7)
n = 400
start = datetime(2026, 8, 1)
rows = []
for i in range(n):
    d = start + timedelta(days=int(rng.randint(0, 70)))
    days = int(rng.choice([2, 3, 5, 6, 7, 7, 8, 9, 10, 12, 15, 21]))
    k = rng.rand()
    date = d if k < 0.70 else (d.strftime("%m/%d/%Y") if k < 0.85 else d.strftime("%b %d, %Y"))
    dur = days if rng.rand() < 0.85 else f"{days} days"
    cat = rng.choice(["Pothole", "pothole", "POTHOLE", "Streetlight", "streetlight", "Noise"])
    rows.append({"request_id": f"R-{5001 + i}", "category": cat,
                 "received": date, "days_to_close": dur})
log = pd.DataFrame(rows)
log = pd.concat([log, log.sample(15, random_state=7)], ignore_index=True)

# QUESTION: how many pothole requests received on or after Sept 1, 2026
# took more than 7 days to close?

def as_date(v):
    if isinstance(v, datetime):
        return v
    if not PARSE_TEXT:
        return pd.NaT          # what a date filter does with text: skips it
    return pd.to_datetime(v, format="mixed")

def as_days(v):
    if isinstance(v, (int, np.integer)):
        return int(v)
    if not PARSE_TEXT:
        return np.nan          # what AVERAGE and COUNTIFS do with text
    return int(str(v).split()[0])

data = log.drop_duplicates() if DEDUPE else log
d = data.assign(date=data.received.map(as_date),
                days=data.days_to_close.map(as_days),
                cat=data.category.str.strip().str.lower())

late = d.days > 7 if STRICT else d.days >= 7
answer = d[(d.cat == "pothole") & (d.date >= datetime(2026, 9, 1)) & late]

print(f"Rows in the log             : {len(log)}")
print(f"Rows after the duplicate step: {len(data)}")
print(f"Dates left unreadable        : {d.date.isna().sum()}")
print(f"Day counts left unreadable   : {d.days.isna().sum()}")
print(f"Average days to close        : {d.days.mean():.2f}")
print(f"\nANSWER: {len(answer)} late pothole requests since Sept 1")
print("\nNothing above raised an error, whatever the settings.")
```

With all three settings on, the answer is 52. Each setting you turn off produces a different answer, and none of them produces a warning. That last line of output is the point of the cell.

**Try changing it:**

1. Set `PARSE_TEXT = False`. The answer drops to 31, and 118 dates and 60 day counts become unreadable. The average barely moves, from 8.30 to 8.49 days. Would you have noticed the missing rows from the average alone?
2. Set `DEDUPE = False`, then `STRICT = False`, one at a time. Which trap changes the answer more? For the `>=` version, which reading does "more than 7 days" actually mean?
3. Describe this log's columns to an AI assistant **without** mentioning that some dates and day counts are text, and ask for a Google Sheets formula for the question. Read the formula: which rows would it skip? Then describe the text values and ask again. What changed in the formula?

## ⚙️ Hands-On: When the Total Points the Wrong Way

Doors 4 and 5 turned on aggregation. This cell compares two tutoring centers that each serve 200 students in intro and advanced courses, then looks at the tutors at one center.

```python
import pandas as pd

# ---- The settings you change ---------------------------------------
SOUTH_ADVANCED_SHARE = 0.80   # share of South's students in advanced courses
T6_STUDENTS          = 8      # how many students tutor T6 has seen
# ---------------------------------------------------------------------

# Two tutoring centers. Each serves 200 students in intro and advanced courses.
# pass rate by course level, and the share of advanced students, per center
centers = {
    "North": {"intro": 0.90, "advanced": 0.40, "adv_share": 0.10},
    "South": {"intro": 0.95, "advanced": 0.50, "adv_share": SOUTH_ADVANCED_SHARE},
}
rows = []
for name, c in centers.items():
    n_adv = round(200 * c["adv_share"])
    for level, n in [("intro", 200 - n_adv), ("advanced", n_adv)]:
        rows.append({"center": name, "level": level, "students": n,
                     "passed": round(n * c[level])})
t = pd.DataFrame(rows)

overall = t.groupby("center")[["students", "passed"]].sum()
overall["pass_rate_%"] = (overall.passed / overall.students * 100).round(1)
print("OVERALL (what a summary slide shows):")
print(overall.to_string(), "\n")

by_level = t.assign(pass_rate_pct=(t.passed / t.students * 100).round(1))
print("BY COURSE LEVEL (what actually happened):")
print(by_level.pivot(index="level", columns="center", values="pass_rate_pct").to_string(), "\n")

# The tutors at South. Who has the lowest pass rate?
tutors = pd.DataFrame({
    "tutor":    ["T1", "T2", "T3", "T4", "T5", "T6"],
    "students": [40, 38, 42, 36, 36, T6_STUDENTS],
    "advanced_share_%": [75, 80, 78, 82, 80, 100],
})
tutors["pass_rate_%"] = (0.95 * (1 - tutors["advanced_share_%"] / 100)
                         + 0.50 * tutors["advanced_share_%"] / 100) * 100
tutors["pass_rate_%"] = tutors["pass_rate_%"].round(1)
print("SOUTH TUTORS:")
print(tutors.sort_values("pass_rate_%").to_string(index=False))
```

North passes 85.0% of its students overall and South only 59.0%, yet South has the higher pass rate in **both** course levels: 95% against 90% in intro, 50% against 40% in advanced. South looks worse only because 80% of its students take advanced courses. At South, tutor T6 has the lowest rate, 50.0%, from 8 students, all of them advanced. Every tutor in this table was built with exactly the same pass rates by level; the ranking is caused entirely by who each tutor was assigned.

**Try changing it:**

1. Set `SOUTH_ADVANCED_SHARE = 0.10`, the same mix as North. South's overall rate becomes 90.5% and the reversal disappears. What does that tell you about what the original overall numbers were measuring?
2. Set `T6_STUDENTS = 80`. The sample is now large, and T6 still ranks last at 50.0%. Does more data make it fair to put T6 on probation? Which of the two problems from Door 5 did the larger sample fix, and which did it leave in place?
3. Paste only the OVERALL table into an AI assistant and ask which center is better. Then paste the BY COURSE LEVEL table and ask again. Both are aggregates, so both are allowed under the AI rule. Does the answer change, and does the assistant explain why?

## ⚙️ In-Class Activity: Debrief — Your Method versus the Model's

About 35 minutes, in the session after the escape room. Work in your escape room teams.

1. **Present first** (3 minutes per team). Before any AI sees the data, each team reports which room it reached, the wrong values it tried, and where each wrong value came from. Your reasoning goes on the record before you see the model's.
2. **Change the rule** (1 minute). Imagine the clinic has an approved, HIPAA-compliant AI tool. Under that policy only, upload the workbook to the AI assistant your team used, and paste this prompt:

   ```
   I am attaching a spreadsheet of visits to two walk-in clinics. The data is in the sheet "Visits"; the sheet "About" describes the columns. Answer the five questions below. For each question: (1) state every assumption you make about the data before computing, especially about duplicate rows, blank or text values, and date formats; (2) write the code you use and run it on the full sheet; (3) report how many rows your calculation included and how many it excluded, and why; (4) give the answer. Do not estimate any number. If you cannot run code on the file, say so and stop.

   1. A visit is valid when it is counted once (some rows are exact copies of another row with the same visit_id; count each visit_id only once), it has a visit date (blank or "TBD" means no date), and it has a recorded wait time (blank or "n/a" means no wait time). How many valid visits are there?
   2. What is the average wait time, in minutes, across all valid visits, rounded to 1 decimal?
   3. How many valid Urgent visits on or after September 1, 2026 had a wait of more than 60 minutes?
   4. The first-visit resolution rate is the percentage of valid visits whose outcome is Resolved. Which clinic performs better? Give the better clinic's resolution rate for Urgent visits, in percent, 1 decimal.
   5. The board says: "Put the doctor with the lowest resolution rate on probation." Which doctor should be put on probation?
   ```

3. **Compare, room by room** (10 minutes). For each question, write down:
   - Did the model compute the answer, or state it?
   - Did it make the same choices you did about duplicates, text wait times, and text dates?
   - Did it flag its assumptions, or choose silently?
   - Who was right where you disagreed, and how do you know?
   - In Room 5, did it name a doctor?
4. **Compare across teams** (5 minutes). Teams used different AI tools. Did they all produce the same numbers? A tool that cannot run code may state numbers anyway; find out which one did.
5. **Write the standing rule** (5 minutes). Each team writes one sentence: a rule for numbers that arrive from an AI tool. The class picks one.

```{note}
A model that runs code will probably answer the first three questions correctly in seconds. Notice why: the prompt spelled out exactly what counts as a valid visit, and that definition came from someone who inspected the data first. On a real job, nobody hands you the definition. Producing it was the work you did in the escape room, and it is the work the model cannot do for you. Questions 4 and 5 are where the models are most likely to differ, because they ask for judgment, not computation.
```

## Part IV — Prompts for Spreadsheet Work

Every prompt below is written to produce **Mode 1** output. Complete versions with verification steps are in Appendix D.6.

**Describe the data before you ask for anything.** This is the prompt that closes the description gap, and it asks the model to find what you missed:

```
I have a Google Sheets log of city service requests, rows 2 through 416. Column A is the request id, B is the category, C is the date received, and D is the number of days to close. I have noticed that some dates in column C are real dates and others are text such as "09/14/2026" or "Sep 14, 2026", that some values in D are text such as "8 days", that the category is typed with mixed capitalization, and that some rows may be exact duplicates. Before writing any formula, list every other problem a column like each of these commonly has that would make a COUNTIFS or AVERAGE give a wrong answer without an error. For each problem, tell me how to check for it in the sheet myself. Do not compute anything.
```

**Get the formula, never the answer:**

```
Write a Google Sheets formula for the following. My data: columns A = ticket id, B = status, C = closed date, D = minutes to resolve, rows 2 through 847. What I want: the median resolution time for tickets whose status is "closed" and whose closed date is on or after 1 September 2026. Return the formula and a one-line explanation of each argument. Do not tell me the answer; I will run it. If the calculation requires an assumption about my data that I have not stated (blank cells, text in a numeric column, date format), state the assumption instead of picking one silently.
```

**Audit a number you were given.** Use this the moment any tool states a figure:

```
You told me the average resolution time is 4.2 days. Give me the exact formula or code that produces that number from my data, written so I can run it myself. Then tell me: how many rows did your calculation include, and were any excluded for any reason? If you did not compute this figure from the data I provided, say so directly.
```

The third prompt is the one that catches fabrications. Run the returned formula; if the result differs from the stated number, you have found one.

## 💡 Example: The Estimate That Was Never Computed

A team lead asked an assistant to review a project spreadsheet and got: *"Based on the data, the team is averaging 1.4x their estimates, so the Q4 milestone should be pushed by approximately three weeks."*

Both numbers were fabricated. The actual ratio was 1.9x, and the three-week figure had no derivation at all: the spreadsheet contained no milestone dates. The recommendation was directionally correct and quantitatively invented, which is the worst combination, because directional correctness is what makes people stop checking.

Asking instead for *the formula that computes the estimate ratio* would have returned something inspectable, run in about four seconds, and produced 1.9.

## 🧭 Reflection

> A fabricated sentence in a document usually reads a little off. A fabricated number in a spreadsheet cell reads exactly like a real one: same font, same alignment, same air of having been calculated.
>
> In the escape room, the AI wrote every formula, and the doors still stayed locked until someone noticed what was in the data. What did your team notice that the AI could not have known, and what did you miss that it could have caught if you had described the data better? Keep your class's standing rule; you will need it in Unit 13.

**Grading.** The Unit 12 grade is the escape room Team Log, scored with the rubric above. The debrief is not graded.

## 📘 Further Reading

- Panko, R. (2008). _Spreadsheet Errors: What We Know._ Journal of End User Computing.
- Bickel, P. J., Hammel, E. A., & O'Connell, J. W. (1975). _Sex Bias in Graduate Admissions: Data from Berkeley._ Science, 187(4175), 398–404.
- McKinney, W. (2022). _Python for Data Analysis_, 3rd Ed., Ch. 7–10. O'Reilly.
- de Castro, L. N., & Ferrari, D. G. (2016). _Introdução à Mineração de Dados_, Cap. 3. Saraiva.
- Dendritic Institute (2025). _AI Literacy Series — Module 2, Part V: AI for Spreadsheets._
