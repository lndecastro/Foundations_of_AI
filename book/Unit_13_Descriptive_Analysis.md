# Unit 13: AI for Descriptive Analysis

> **Sessions 19–20** · Oct 21, 26 · **HW11 assigned Oct 28, due Nov 04**

Part V is where the course turns back toward computing. Units 10 through 12 used AI on documents; Units 13 through 15 use it on data and code. The skills are related but the stakes rise, because a wrong number in an analysis propagates into every conclusion drawn from it.

**Exploratory data analysis** is what you do before modeling: look at the data, understand its shape, find what is broken. Unit 3 told you preprocessing consumes the largest share of effort in an AI project. This unit is that share.

The specific opportunity is that AI is genuinely excellent at the mechanical parts of EDA — generating the right summary code, suggesting what to check next, explaining an unfamiliar statistic — and genuinely poor at the part that matters, which is deciding what the numbers mean.

## Learning Objectives

After completing this unit, you will be able to:

- Describe where EDA fits in the data science workflow and why it precedes modeling.
- Compute and **interpret** distributions, central tendency, dispersion, and association.
- Use AI to accelerate EDA while retaining responsibility for every claim.
- Recognize when a summary statistic conceals more than it reveals.

## Part I — What EDA Is and Is Not

Exploratory data analysis, in Tukey's original framing, is detective work: you approach data without a hypothesis to confirm, looking for what is there.

**EDA is:** examining distributions, spotting anomalies and missing values, discovering relationships, forming hypotheses, and deciding whether the data can answer the question at all.

**EDA is not:** confirming what you already believe, generating charts to decorate a report, or a step to skip when the deadline is close.

```{note}
The most valuable outcome of EDA is sometimes the conclusion that **the data cannot answer the question**. This is a real result and a professionally difficult one to deliver. It is also the outcome an AI assistant is least likely to volunteer, because "here is an analysis" is a far more typical continuation than "you should not proceed."
```

### 1.1 Where It Fits

| Stage | Question |
| :--- | :--- |
| Problem definition | What are we actually trying to decide? |
| Data collection | What do we have, and where did it come from? |
| **EDA** | **What is in this data, and what is wrong with it?** |
| Preprocessing | Clean, transform, engineer features |
| Modeling | Fit and evaluate (Units 3–5) |
| Communication | Make the result actionable (Unit 14) |

## Part II — The Measures, and What They Hide

### 2.1 Central Tendency

- **Mean** — the balance point. Sensitive to extreme values.
- **Median** — the middle. Robust to extremes.
- **Mode** — the most frequent. The only one that works for nominal data.

When mean and median diverge substantially, the distribution is skewed, and reporting the mean alone is a choice with consequences. Salary data is the standard example: the mean is pulled upward by a few large values, so "average salary" and "typical salary" are different numbers, and which one gets reported is often a decision about what the reader should conclude.

### 2.2 Dispersion

Central tendency without dispersion is nearly useless. Two datasets with identical means can be entirely different.

- **Range** — max minus min. One outlier controls it.
- **IQR** — the middle 50%. Robust.
- **Variance / Standard deviation** — average squared / typical deviation from the mean.
- **Coefficient of variation** — standard deviation relative to the mean, which permits comparison across different units.

### 2.3 Shape

- **Skewness** — asymmetry. Positive means a long right tail.
- **Kurtosis** — tail weight. High kurtosis means extreme values are more common than a normal distribution would predict.

Kurtosis matters more than its obscurity suggests: a model built assuming normally distributed errors will be surprised, expensively, by a heavy-tailed reality.

### 2.4 Association

- **Covariance** — do two variables move together? Unbounded and unit-dependent, therefore hard to interpret directly.
- **Pearson correlation** — covariance normalized to [-1, 1]. Measures **linear** association only.
- **Spearman correlation** — the same on ranks. Catches monotonic but non-linear relationships.

```{warning}
A Pearson correlation near zero does **not** mean no relationship. It means no *linear* relationship. A perfect parabola has a correlation of approximately zero. Unit 14 shows you four datasets with identical correlations and utterly different structures.
```

## ⚙️ Hands-On: A Complete Descriptive Pass

We use the diabetes dataset bundled with scikit-learn — 442 patients, ten baseline measurements, and a quantitative disease-progression score one year later.

```python
import numpy as np
import pandas as pd
from scipy import stats
from sklearn.datasets import load_diabetes

d = load_diabetes(as_frame=True)
df = d.frame.rename(columns={"target": "progression"})

print(f"Shape: {df.shape[0]} rows × {df.shape[1]} columns")
print(f"Missing values: {df.isna().sum().sum()}\n")

# --- Central tendency, dispersion, shape --------------------------
summary = pd.DataFrame({
    "mean":     df.mean(),
    "median":   df.median(),
    "std":      df.std(),
    "IQR":      df.quantile(.75) - df.quantile(.25),
    "skew":     df.skew(),
    "kurtosis": df.kurtosis(),
}).round(3)
print(summary.to_string())

# --- Where mean and median disagree most --------------------------
gap = ((df.mean() - df.median()).abs() / df.std()).sort_values(ascending=False)
print(f"\nLargest mean-median divergence: {gap.index[0]} ({gap.iloc[0]:.3f} SDs)")

# --- Association: linear vs. monotonic ----------------------------
print("\nCorrelation with progression:")
rows = []
for col in df.columns.drop("progression"):
    r, _ = stats.pearsonr(df[col], df.progression)
    rho, _ = stats.spearmanr(df[col], df.progression)
    rows.append((col, r, rho, rho - r))
assoc = pd.DataFrame(rows, columns=["variable", "pearson", "spearman", "diff"])
print(assoc.sort_values("pearson", key=abs, ascending=False)
          .round(3).to_string(index=False))
```

### Reading What You Just Produced

Three things are worth noticing in this output.

**The features are pre-standardized.** Every mean is essentially zero and every standard deviation nearly identical. That is not a property of the patients; it is a preprocessing decision made by whoever prepared this dataset. **You cannot recover the original units from this file.** A blood pressure of `0.021` is meaningless to a clinician, and any analysis you present in these units is uninterpretable to the people who would act on it.

This is the kind of thing EDA exists to catch, and it is invisible if you go straight to modeling.

**The `diff` column** shows where Spearman and Pearson disagree. A large gap signals a monotonic but non-linear relationship — the variable matters, but not in a straight line.

**Nothing here tells you what to do.** The numbers are facts about the file. Whether the strongest correlate is clinically actionable, confounded, or an artifact of how patients entered this study is not in the data, and no amount of further computation will put it there.

**Try changing it:**

1. Which variable has the largest `|skew|`? Plot its histogram. Would you report its mean?
2. Compute the coefficient of variation for each column. Why is it useless here, and what does that tell you about the standardization?
3. Take the top correlate and compute its correlation with **every other feature**. If it correlates strongly with several, what does that do to a claim that it "drives" progression?

## Part III — Using AI in EDA Without Losing the Plot

The Unit 12 distinction governs everything here: **prefer code you can run over numbers you are told.**

### 3.1 Prompts That Work

**Generate the analysis, not the answer:**

> *Here are the column names and dtypes of a dataset about [domain]. Write pandas code that produces a descriptive summary: shape, missing counts, central tendency, dispersion, skewness, and a correlation matrix. Do not interpret the results. Return only code.*

**Ask what to check next:**

> *Given this summary output [paste], what are the five most important things I should check before modeling? For each, state what problem it would reveal. Do not tell me what the data means.*

**Explain an unfamiliar measure:**

> *Explain kurtosis to someone who understands standard deviation. Give a concrete case where high kurtosis would break a model that assumed normality.*

**Challenge your own reading:**

> *I have concluded that [claim] from this output. Give me the three strongest reasons that conclusion could be wrong.*

The last one is the highest-value prompt in this unit. Your default failure mode in EDA is finding the pattern you were hoping for, and a system with no stake in your hypothesis is a useful adversary.

### 3.2 Prompts That Fail

> *"Analyze this data and tell me what's interesting."*

You will get something. It will be fluent, it will be structured, and it will be the analysis that is *typical* for data shaped like yours rather than the analysis of your data. Worse, it will not distinguish claims it computed from claims it inferred from the column names.

```{warning}
An assistant given column names alone will readily produce plausible findings about relationships in data it has never seen. "Age likely correlates positively with progression" is a statement about the world, not about your file, and it will be phrased identically to a statement about your file.

When an AI reports a finding, ask one question: **what computation produced this?** If there is no answer, it is a prior, not a result.
```

## ⚙️ Hands-On: The Adversarial Reading

Every EDA conclusion should survive a challenge. This cell builds one for you to run against your own claims.

```python
import numpy as np
import pandas as pd

rng = np.random.RandomState(3)
n = 300

# Two groups measured by an instrument that was recalibrated mid-study
group = rng.choice(["A", "B"], n, p=[.5, .5])
phase = np.where(np.arange(n) < 150, "before", "after")
true_effect = np.where(group == "A", 0.0, 0.15)
calibration = np.where(phase == "before", 0.0, 0.9)      # the confound
value = 10 + true_effect + calibration + rng.normal(0, .4, n)

df = pd.DataFrame({"group": group, "phase": phase, "value": value})

print("NAIVE READING")
print(df.groupby("group").value.agg(["mean", "std", "count"]).round(3).to_string())

print("\nSTRATIFIED BY PHASE")
print(df.groupby(["phase", "group"]).value.agg(["mean", "count"])
        .round(3).to_string())

print("\nGROUP BALANCE ACROSS PHASES")
print(pd.crosstab(df.phase, df.group).to_string())
```

The naive reading shows a difference between groups. The stratified reading shows that almost all of it comes from the recalibration, not the group — and the crosstab shows whether the groups were balanced across phases in the first place.

This is **Simpson's paradox** territory, and it is not exotic. It is what happens whenever a variable you did not think to record is doing the work. No summary statistic can warn you; only the question *"what else changed?"* can, and asking it is your job.

**Try changing it:**

1. Set `true_effect` to zero for both groups. The naive reading still shows a difference. What produced it?
2. Change the group assignment so that group B is over-represented in the "after" phase. How much worse does the naive reading get?
3. Ask an AI assistant to interpret only the **naive** output. Does it warn you about confounding, or does it explain the difference?

Question 3 is the exercise. An assistant shown one table will interpret that table.

## 💡 Example: The Analysis That Should Not Have Proceeded

A student analyzing course-evaluation data found a correlation of 0.61 between class size and average rating, and prepared a recommendation to cap enrollment.

EDA would have surfaced three things first. Large classes in this dataset were disproportionately introductory courses. Introductory courses were disproportionately taught by first-year instructors. And response rates fell from 68% in small classes to 31% in large ones — meaning the large-class ratings represented a self-selected third of students.

The correlation was real. The recommendation did not follow from it. Nothing in the summary statistics would have stopped the student; only the habit of asking what else varies alongside the variable of interest.

## 🧭 Reflection

> You can now generate a complete descriptive summary of an unfamiliar dataset in under a minute, and ask an assistant to explain any statistic in it.
>
> Which of the three failures in the example above could that speed have caught — and which required knowing something that was not in the file at all?

**Connecting to HW11 (Data Analysis, Units 13–14):** assigned Oct 28, due Nov 04. Perform a full descriptive pass on a dataset of your choosing. Submit: the code (Mode 1 throughout), your summary output, three findings, and — carrying equal weight — **one confound or limitation you identified that the statistics alone would not have revealed**. State every prompt used and disclose all AI assistance.

## 📘 Further Reading

- Tukey, J. W. (1977). _Exploratory Data Analysis_. Addison-Wesley.
- de Castro, L. N., & Ferrari, D. G. (2016). _Introdução à Mineração de Dados_, Cap. 3. Saraiva.
- Wickham, H., & Grolemund, G. (2023). _R for Data Science_, 2nd Ed., Ch. 10–12. O'Reilly.
- Dendritic Institute (2025). _Data Analysis with AI — Module 3: Descriptive Analysis._
