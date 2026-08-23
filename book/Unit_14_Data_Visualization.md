# Unit 14: AI for Data Visualization

```text
Session 21 | Oct 28 | HW11 assigned Oct 28, due Nov 04
```

Unit 13 produced numbers. This session is about what numbers cannot show you, and about the fact that the same data can be drawn to support opposite conclusions without a single value being falsified.

Visualization is the point in the pipeline where analysis becomes persuasion, and the transition is easy to make without noticing. That makes it the place where honesty has to be a deliberate practice rather than a default.

## Learning Objectives

After completing this unit, you will be able to:

- Demonstrate why summary statistics require visual inspection.
- Select an appropriate chart type from the shape of the question.
- Apply design principles that reduce cognitive load rather than decorate.
- Identify how a technically accurate chart can mislead, including your own.

## Part I — Why You Must Look

### 1.1 The Case That Settles It

Four datasets. Nearly identical mean, variance, correlation, and regression line. Completely different structures. This is **Anscombe's Quartet**, and it is the strongest argument in statistics for a single practice: plot the data.

```python
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
from scipy import stats

ans = sns.load_dataset("anscombe")   # if offline, see the fallback note below

print(f"{'set':>4}{'mean x':>9}{'mean y':>9}{'std y':>8}"
      f"{'corr':>8}{'slope':>8}{'intercept':>11}")
for name, g in ans.groupby("dataset"):
    lr = stats.linregress(g.x, g.y)
    print(f"{name:>4}{g.x.mean():9.2f}{g.y.mean():9.2f}{g.y.std():8.2f}"
          f"{g.x.corr(g.y):8.3f}{lr.slope:8.3f}{lr.intercept:11.3f}")

fig, axes = plt.subplots(2, 2, figsize=(10, 8))
for ax, (name, g) in zip(axes.flat, ans.groupby("dataset")):
    ax.scatter(g.x, g.y, s=45)
    lr = stats.linregress(g.x, g.y)
    xs = np.linspace(3, 20, 10)
    ax.plot(xs, lr.intercept + lr.slope * xs, "r-", lw=1)
    ax.set_title(f"Dataset {name}"); ax.set_xlim(3, 20); ax.set_ylim(2, 14)
fig.suptitle("Identical statistics. Four different realities.",
             fontsize=13, weight="bold")
plt.tight_layout(); plt.show()
```

Read the table first, then the plot. The table says these are the same data. They are not: one is a clean linear relationship, one is a perfect curve, one is a tight line ruined by a single outlier, and one has all its x-values identical except for one point that single-handedly creates the entire correlation.

```{note}
**Offline fallback.** If `sns.load_dataset` cannot reach the network, the quartet is small enough to type: the four x/y series are published in Anscombe (1973) and reproduced in the Dendritic *Data Analysis with AI* materials, Module 4.
```

**Try changing it:**

1. Remove the single outlier from Dataset III and recompute the correlation. How much of the "relationship" was that one point?
2. In Dataset IV, what would you have to know about data collection to explain that structure? Notice the answer is not in the file.
3. Ask an AI assistant to interpret the **summary table alone**. Does it conclude the four datasets are equivalent?

### 1.2 Preattentive Processing

Some visual properties are processed before conscious attention — in under 250 milliseconds, without search. Position, length, color hue, size, orientation, and enclosure are among them.

This has one practical consequence worth internalizing: **encode your most important variable in position or length**, because those are the channels humans read most accurately. Encode secondary variables in color or size. Do not encode anything important in a channel people read poorly, such as area or angle — which is precisely the problem with pie charts.

## Part II — Choosing the Chart

The selection is determined by the **question**, not by the data type and certainly not by which chart looks best.

| Your question | Chart |
| :--- | :--- |
| How is one variable distributed? | Histogram, density plot, box plot |
| How do distributions compare across groups? | Grouped box or violin plot |
| Do two quantitative variables relate? | Scatter plot |
| Do many variables relate? | Scatterplot matrix, correlation heatmap |
| How do amounts compare across categories? | Bar chart |
| How do parts make a whole? | Stacked bar (rarely a pie) |
| How does something change over time? | Line chart |
| How does something vary over space? | Choropleth, bubble map |
| How does something flow between states? | Sankey diagram |

```{warning}
**Pie charts.** They encode value in angle and area, the two channels humans read least accurately, and they fail entirely beyond about four categories. A bar chart answers the same question better in nearly every case. Use a pie chart when you have two or three categories and the point is "roughly half" — and be aware that you are choosing familiarity over precision.
```

## ⚙️ Hands-On: The Same Data, Two Honest Charts

No number is falsified in either chart below. They support different conclusions.

```python
import numpy as np
import matplotlib.pyplot as plt

months = np.arange(1, 13)
incidents = np.array([42, 44, 41, 43, 45, 42, 40, 41, 39, 40, 38, 37])

fig, ax = plt.subplots(1, 2, figsize=(13, 5))

# --- Chart A: truncated axis -------------------------------------
ax[0].plot(months, incidents, "o-", color="crimson", lw=2)
ax[0].set_ylim(36, 46)
ax[0].set_title("Incidents Down 12% — Reliability Push Working",
                weight="bold")
ax[0].set_xlabel("Month"); ax[0].set_ylabel("Incidents")
ax[0].grid(alpha=.3)

# --- Chart B: zero baseline --------------------------------------
ax[1].plot(months, incidents, "o-", color="steelblue", lw=2)
ax[1].set_ylim(0, 50)
ax[1].set_title("Incident Volume Broadly Flat Across the Year",
                weight="bold")
ax[1].set_xlabel("Month"); ax[1].set_ylabel("Incidents")
ax[1].grid(alpha=.3)

plt.tight_layout(); plt.show()

drop = (incidents[0] - incidents[-1]) / incidents[0] * 100
print(f"Absolute change : {incidents[0]} → {incidents[-1]} "
      f"({incidents[0] - incidents[-1]} incidents)")
print(f"Relative change : {drop:.1f}%")
print(f"Monthly std dev : {incidents.std():.1f}  "
      f"(is the trend larger than the noise?)")
```

Both titles are defensible. Both axes are labeled. The 12% figure is arithmetically correct. And the standard deviation tells you that the month-to-month noise is comparable to the entire "trend," which neither chart communicates and which is the single most decision-relevant fact available.

```{warning}
The truncated axis is not always dishonest — for data with a meaningful non-zero baseline, such as body temperature, starting at zero would be absurd. The question is never "is the axis truncated" but **"does the visual magnitude of the change match its real importance?"**

You will be tempted by Chart A. It will be for a result you worked hard on, and it will feel like clarity rather than exaggeration. That is what makes this the hardest habit in the unit.
```

**Try changing it:**

1. Add error bars of ±1 standard deviation to Chart A. Does the story survive?
2. Extend the series backwards with three years of similar values. Is a 12% within-year change unusual?
3. Which chart would you show a manager who is deciding whether to continue funding the reliability push — and can you defend that choice as an analyst rather than an advocate?

## Part III — Using AI for Visualization

AI is strong at chart **implementation** and weak at chart **judgment**, which mirrors every other unit in Part IV and V.

**Effective prompts:**

```text
Write matplotlib code for the following.

Question the chart must answer: <do resolution times differ across severity levels?>
Data: <~800 rows; "severity" is one of four ordered categories; "minutes" is heavily right-skewed with outliers I want visible, not clipped>

Rules: label both axes with units. Do not truncate the y-axis. Do not add a title that merely restates the axis labels. Return only code.
```

```text
Below is the code for a chart I built. I am claiming <your claim> from it.

You are a skeptical reviewer. What could you legitimately object to about how this is drawn? Check specifically: axis truncation, missing baseline, unlabeled units, a selected date range, aggregation that hides variance, and colour choices that imply a judgment.

For each objection, say how I would fix it and what I would lose by fixing it.

CODE: <paste>
```

```text
I want to answer <your question> from data with columns <list them>.

Suggest three different chart types. For each: what it would reveal, what it would hide, and which audience it suits.

Do not recommend one. I will choose.
```

**Ineffective prompts:**

> *"Make a nice visualization of this data."*

You get a default chart — usually a bar chart, usually with a title restating the axis labels. Defaults are not answers to questions.

```{note}
The second prompt above — asking for objections to your own chart — is the most useful of the three. It reliably surfaces truncated axes, missing baselines, unlabeled units, and cherry-picked ranges, and it does so without the emotional investment you have in the result. It is the visualization equivalent of the adversarial reading from Unit 13.
```

## Part IV — From Chart to Story

A dashboard full of accurate charts can still fail, because the reader does not know where to look or what to do.

**Three questions before publishing any chart or dashboard:**

1. **Who reads this, and what decision do they make?** A chart for an engineer debugging a regression and a chart for a director allocating budget are different charts, even from identical data.
2. **What is the one thing they should take away?** If you cannot say it in a sentence, the chart is not finished.
3. **What would change their mind?** A visualization that cannot be contradicted by the data is decoration.

For dashboards specifically: order by decision priority, not by data availability. Most inherited dashboards are ordered by what was easy to query, which is why nobody looks at them.

## 💡 Example: The Chart That Ended the Argument

A team argued for months about whether their API was getting slower. Both sides had charts. One showed mean latency, flat. The other showed p99 latency, rising sharply.

Both were correct. The mean was flat because most requests were unaffected. The p99 was rising because a growing subset — the users with the largest accounts, and the ones most likely to churn — were experiencing severe degradation.

The chart that ended the argument was a distribution plot per month, showing a bimodal shape emerging over time. No summary statistic showed it. Anscombe's lesson, in production.

## 🧭 Reflection

> You will build a chart this semester for your capstone, and you will want it to support your conclusion.
>
> What specific check will you run on it before presenting — and would you actually run that check on a chart that came out in your favor?

**HW11 (Data Analysis, Units 13–14) is due Nov 04.** Submit the descriptive pass from Unit 13, plus **three charts** with a written justification of each chart-type choice, plus one chart drawn **two defensible ways** with an argument for which you would publish and why. State every prompt used.

## 📘 Further Reading

- Anscombe, F. J. (1973). _Graphs in Statistical Analysis._ The American Statistician, 27(1).
- Tufte, E. (2001). _The Visual Display of Quantitative Information_, 2nd Ed. Graphics Press.
- Wilke, C. (2019). _Fundamentals of Data Visualization_. O'Reilly.
- Cairo, A. (2019). _How Charts Lie_. W. W. Norton.
- Dendritic Institute (2025). _Data Analysis with AI — Modules 4, 5 and 7._
