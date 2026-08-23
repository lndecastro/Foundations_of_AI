# Unit 12: Spreadsheets

```text
Session 18 | Oct 19 | HW10 (Office Portfolio) due Oct 21
```

Spreadsheets are where AI assistance is most useful and most dangerous, and the two facts have the same cause: **a spreadsheet looks authoritative.** A number in a cell carries an air of computation whether or not any computation occurred.

This session closes Part IV and sets up Part V. The habits you build here — never trusting a number you did not verify, always separating computation from generation — are the habits that make Units 13 and 14 safe.

## Learning Objectives

After completing this unit, you will be able to:

- Distinguish AI that **writes a formula** from AI that **states a number**, and prefer the first.
- Generate and audit formulas, pivots, and cleaning scripts.
- Detect fabricated arithmetic in AI output.
- Build a small analysis with a verification step at every stage.

## Part I — The Central Distinction

There are two fundamentally different things an AI tool can do with your data, and confusing them is the source of nearly every serious error in this domain.

**Mode 1 — It writes code or a formula that computes the answer.**
You get `=SUMIFS(D:D, B:B, "closed", C:C, ">="&DATE(2026,9,1))` or a few lines of pandas. The computation happens in your spreadsheet or your runtime, deterministically. The AI is generating a *specification for arithmetic*, and if the specification is wrong you can read it and see that it is wrong.

**Mode 2 — It reads your data and tells you the answer.**
You get "the average resolution time was 4.2 days." This number was **generated as text**, by a system whose objective is producing likely continuations. Unit 5's argument applies with full force: it may be right, it may be approximately right, and it may be entirely invented — and it will look identical in all three cases.

```{warning}
**Prefer Mode 1 always.** When the answer arrives as a formula you can inspect and re-run, you have an auditable artifact. When it arrives as a sentence, you have a claim.

This single habit prevents more spreadsheet errors than everything else in this unit combined. If a tool gives you a number, ask it for the formula that produced the number — and if it cannot produce one, you have learned something important about where the number came from.
```

## Part II — Where AI Genuinely Helps

**Formula generation.** Describing what you want in words and receiving a formula is dramatically faster than remembering `INDEX/MATCH` argument order. It is also easy to verify: run it on rows where you know the answer.

**Data cleaning.** Inconsistent dates, mixed units, stray whitespace, names in three formats. AI-generated cleaning code is genuinely good at this, and the results are checkable by inspection.

**Structure explanation.** Handed an inherited spreadsheet with forty columns and no documentation, "what does this file appear to contain and which columns look like keys" is a fast orientation.

**Formula debugging.** A `#REF!` in a nested formula is exactly the kind of syntactic puzzle these tools handle well.

**Pivot and aggregation design.** Describing the summary you want and getting the grouping logic.

### 2.1 Engineering Spreadsheets

The examples in this session use data engineers actually keep:

- **Sprint velocity** — story points committed vs. completed, by sprint.
- **Test coverage** — coverage by module over time, with a threshold gate.
- **Incident log** — severity, duration, time to detect, time to resolve.
- **Effort estimation** — estimated vs. actual hours, and the ratio between them.
- **Build times** — duration by pipeline stage, tracked for regression.

## ⚙️ Hands-On: Catching a Fabricated Number

This is the most important exercise in Part IV. We build a small dataset, compute the truth, and then examine what happens when a number arrives without a computation behind it.

```python
import pandas as pd
import numpy as np

rng = np.random.RandomState(11)
n = 60
sprint_log = pd.DataFrame({
    "sprint":    np.repeat([f"S{i:02d}" for i in range(1, 13)], 5),
    "engineer":  np.tile(["ana", "bo", "cai", "dee", "eli"], 12),
    "committed": rng.randint(3, 13, n),
})
sprint_log["completed"] = np.maximum(
    0, sprint_log["committed"] - rng.binomial(3, 0.45, n)
)
sprint_log["hours_est"] = sprint_log["committed"] * rng.uniform(1.5, 3.0, n).round(1)
sprint_log["hours_act"] = (sprint_log["hours_est"]
                           * rng.uniform(0.8, 2.1, n)).round(1)

print(sprint_log.head(8).to_string(index=False))

# --- MODE 1: the computation is visible and re-runnable ------------
by_sprint = (sprint_log
             .groupby("sprint")
             .agg(committed=("committed", "sum"),
                  completed=("completed", "sum"))
             .assign(rate=lambda d: (d.completed / d.committed * 100).round(1)))

print("\nCompletion rate by sprint (Mode 1 — auditable):")
print(by_sprint.to_string())

overall = sprint_log.completed.sum() / sprint_log.committed.sum() * 100
ratio   = (sprint_log.hours_act / sprint_log.hours_est).mean()
worst   = by_sprint.rate.idxmin()

print(f"\nOverall completion rate : {overall:.1f}%")
print(f"Mean actual/estimate    : {ratio:.2f}x")
print(f"Worst sprint            : {worst} ({by_sprint.rate.min():.1f}%)")

# --- THE AUDIT -----------------------------------------------------
# Now paste the first 8 rows above into an AI assistant and ask, in words:
#   "What is the overall completion rate and the mean actual/estimate ratio?"
# Then compare its answer to the numbers this cell computed.
print("\n" + "=" * 66)
print("AUDIT: ask an assistant the same questions in plain language,")
print("using ONLY the 8 rows printed above. Then answer:")
print("  1. Did it compute, or did it state?")
print("  2. Is its number right for 8 rows? For all 60?")
print("  3. Did it tell you which subset it used?")
print("=" * 66)
```

The third audit question is the one that catches people. Given a partial view of the data, an assistant will often answer as though it had all of it, without flagging that it was working from an excerpt. The number it gives is not a lie about arithmetic — it is a correct computation over the wrong population, presented without the qualifier that would have made it useful.

**Try changing it:**

1. Ask an assistant for the completion rate, then ask it for **the formula or code** that computes it. Run that. Do the two agree?
2. Change one `committed` value to `0`. Ask for the rate. Did it handle division by zero, or produce something confident and wrong?
3. Ask "which engineer is underperforming?" Notice that the question has no defensible answer from this data — five engineers, twelve sprints, no context on task difficulty. Does the assistant say so, or does it name someone?

Question 3 is the responsibility beat. A tool that names an underperforming employee from noise has produced a judgment about a person from insufficient evidence, and someone might act on it.

## ⚙️ Hands-On: Cleaning, the Task AI Is Actually Good At

```python
import pandas as pd

messy = pd.DataFrame({
    "ticket":   ["T-1001", "t-1002", "T1003", " T-1004", "T-1005"],
    "opened":   ["2026-03-01", "03/02/2026", "Mar 3, 2026", "2026-03-04", ""],
    "severity": ["High", "high", "HIGH", "Sev-1", "medium"],
    "minutes":  ["45", "1h20m", "90", "", "2 hours"],
})
print("BEFORE:\n", messy.to_string(index=False))

# Cleaning logic of the kind an assistant generates well.
# Read every line before running it — that is the whole point.
clean = messy.copy()
clean["ticket"] = (clean.ticket.str.strip().str.upper()
                   .str.replace(r"^T-?", "T-", regex=True))
clean["opened"] = pd.to_datetime(clean.opened, format="mixed", errors="coerce")

sev_map = {"high": "High", "sev-1": "High", "medium": "Medium", "low": "Low"}
clean["severity"] = clean.severity.str.strip().str.lower().map(sev_map)

def to_minutes(v):
    v = str(v).strip().lower()
    if not v:
        return None
    if v.isdigit():
        return int(v)
    import re
    h = re.search(r"(\d+)\s*h", v)
    m = re.search(r"(\d+)\s*m", v)
    if "hour" in v:
        h = re.search(r"(\d+)", v)
    total = (int(h.group(1)) * 60 if h else 0) + (int(m.group(1)) if m else 0)
    return total or None

clean["minutes"] = clean.minutes.apply(to_minutes)

print("\nAFTER:\n", clean.to_string(index=False))
print("\nUnparsed values (always check these):")
print(f"  opened   : {clean.opened.isna().sum()}")
print(f"  severity : {clean.severity.isna().sum()}")
print(f"  minutes  : {clean.minutes.isna().sum()}")
```

The final block is not decoration. **Cleaning code fails silently** — a date it cannot parse becomes `NaT`, a severity it does not recognize becomes `NaN`, and your analysis proceeds over a dataset quietly missing rows. Always count what did not survive.

**Try changing it:**

1. Add a row with severity `"Sev-2"`. It becomes `NaN`. Would you have noticed without the count?
2. Add `"1h20m"` handling for `"1:20"`. Does your fix break any existing case?
3. Ask an assistant to write this cleaner from the messy data alone. Compare its handling of edge cases against this version. What did it silently drop?

## Part III — Prompts for Spreadsheet Work

Every prompt below is written to produce **Mode 1** output. Complete versions with verification steps are in Appendix D.6; these are the two you will use most.

**Get the formula, never the answer:**

```text
Write a Google Sheets formula for the following. My data: columns A = ticket id, B = status, C = closed date, D = minutes to resolve, rows 2 through 847. What I want: the median resolution time for tickets whose status is "closed" and whose closed date is on or after 1 September 2026. Return the formula and a one-line explanation of each argument. Do not tell me the answer — I will run it. If the calculation requires an assumption about my data that I have not stated (blank cells, text in a numeric column, date format), state the assumption instead of picking one silently.
```

**Audit a number you were given** — use this the moment any tool states a figure:

```text
You told me the average resolution time is 4.2 days. Give me the exact formula or code that produces that number from my data, written so I can run it myself. Then tell me: how many rows did your calculation include, and were any excluded for any reason? If you did not compute this figure from the data I provided, say so directly.
```

The second prompt is the one that catches fabrications. Run the returned code; if the result differs from the stated number, you have your HW10 evidence.

## 💡 Example: The Estimate That Was Never Computed

A team lead asked an assistant to review a project spreadsheet and got: *"Based on the data, the team is averaging 1.4x their estimates, so the Q4 milestone should be pushed by approximately three weeks."*

Both numbers were fabricated. The actual ratio was 1.9x, and the three-week figure had no derivation at all — the spreadsheet contained no milestone dates. The recommendation was directionally correct and quantitatively invented, which is the worst combination, because directional correctness is what makes people stop checking.

Asking instead for *the formula that computes the estimate ratio* would have returned something inspectable, run in about four seconds, and produced 1.9.

## 🧭 Reflection

> A fabricated sentence in a document usually reads a little off. A fabricated number in a spreadsheet cell reads exactly like a real one — same font, same alignment, same air of having been calculated.
>
> Given that, what standing rule would you adopt for numbers that arrive from an AI tool? Write it down; you will need it in Unit 13.

**HW10 (Office Portfolio) is due Oct 21.** Submit across all three Part IV units: two documents from Unit 10 with prompts, raw output, edits, and rationale; the capstone outline and adversarial questions from Unit 11; and from Unit 12, **one analysis of real data** — your own, or the sprint log above — including at least one instance where you caught an AI-produced number that was wrong, with the evidence. If you did not catch one, say so and explain how you checked.

## 📘 Further Reading

- Panko, R. (2008). _Spreadsheet Errors: What We Know._ Journal of End User Computing.
- McKinney, W. (2022). _Python for Data Analysis_, 3rd Ed., Ch. 7–10. O'Reilly.
- de Castro, L. N., & Ferrari, D. G. (2016). _Introdução à Mineração de Dados_, Cap. 3. Saraiva.
- Dendritic Institute (2025). _AI Literacy Series — Module 2, Part V: AI for Spreadsheets._
