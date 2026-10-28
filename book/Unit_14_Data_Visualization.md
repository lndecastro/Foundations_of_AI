# Unit 14: AI for Data Visualization

```
Session 21 | Oct 28 | HW11 assigned Oct 28, due Nov 04
```

Unit 13 produced numbers. This session is about what numbers cannot show you, and about the fact that the same data can be drawn to support opposite conclusions without a single value being falsified.

Visualization is the point in the pipeline where analysis becomes persuasion, and the transition is easy to make without noticing. That makes it the place where honesty has to be a deliberate practice rather than a default.

## Learning Objectives

After completing this unit, you will be able to:

- Demonstrate why summary statistics require visual inspection.
- Select an appropriate chart type from the shape of the question.
- Build the main chart types of six families (distributions, associations, amounts, proportions, evolution and flow, geospatial) in Python and with an AI assistant, and evaluate the differences between the two.
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

The selection is determined by the **question**, not by the data type and certainly not by which chart looks best. Every chart in this unit belongs to one of six families, and each family answers one kind of question.

| Your question | Family | Charts |
| :--- | :--- | :--- |
| How is one variable distributed? How do distributions compare across groups? | Distributions | Histogram, boxplot, violin plot |
| Do two or more quantitative variables relate? | Associations | Scatter plot, bubble chart, scatterplot matrix, heatmap |
| How do amounts compare across categories or profiles? | Amounts | Bar chart (simple, grouped), radar chart |
| How do parts make a whole? | Proportions | Bar chart, pie, doughnut, treemap, sunburst |
| How does something change over time, or flow between states? | Evolution and flow | Line chart, Sankey diagram, Gantt chart |
| How does something vary over space? | Geospatial | Choropleth map, bubble map |

```{warning}
**Pie charts.** They encode value in angle and area, the two channels humans read least accurately, and they fail entirely beyond about four categories. A bar chart answers the same question better in nearly every case. Use a pie chart when you have two or three categories and the point is "roughly half" — and be aware that you are choosing familiarity over precision.
```

## Part III — Six Chart Families: Code, Then Prompt

This part is a series of six short rounds, one per family. Each round follows the same four steps, so that the class stops only once per family:

1. **Concept.** The family is explained: what each chart is for, what data it needs, and how to read it.
2. **Code.** You run the family's single Python cell in Colab and read the printed numbers.
3. **Prompt.** You upload the CSV file(s) named in the round to an AI assistant that can run code (ChatGPT, Claude, or Gemini) and paste the prompt exactly as written.
4. **Compare.** You put the Python charts and the AI charts side by side and work through the comparison checklist.

The point is not to decide which one "wins." The Python cell is a set of deliberate choices written down; the AI output is a set of choices **someone else made for you**. Your job is to find those choices.

### How to Compare: The Five Checks

Use these five checks in every round. Each round adds the specific things to look for.

| Check | Question to ask |
| :--- | :--- |
| **Numbers** | Do the values the AI printed or plotted match the values printed by the Python cell? If not, which one is right, and why do they differ? |
| **Encoding** | Did the AI map the same variables to the same channels (position, length, color, size)? |
| **Defaults** | What did the AI decide that the prompt did not specify — bins, scales, colors, sort order, axis ranges, dropped rows? |
| **Errors** | Is anything wrong: a mislabeled axis, a silently dropped group, a statistic computed with a different convention? |
| **Judgment** | Does the AI's written interpretation say something the chart does not support? |

### Cell 0 — Prepare the Data Files for the AI

Run this cell once at the start of the session. It saves the seven datasets used in Part III as CSV files and downloads them to your computer, so you can upload them to the AI assistant in each round. Colab may ask permission to download multiple files; allow it. If a download is blocked, open the folder icon in Colab's left sidebar and download the files from there.

```python
import numpy as np
import pandas as pd
import seaborn as sns
import plotly.express as px

# --- 1. Synthetic distributions (same seed as the Distributions cell) ---
rng = np.random.default_rng(42)
n = 2000
dists = {
    "Normal":       rng.normal(0, 1, n),
    "Uniform":      rng.uniform(0, 1, n),
    "Laplace":      rng.laplace(0, 1, n),
    "Right-skewed": rng.beta(1.5, 8, n),
    "Bimodal":      np.concatenate([rng.normal(-2, 0.6, n // 2),
                                    rng.normal(2, 0.6, n // 2)]),
}
pd.DataFrame([(k, v) for k, arr in dists.items() for v in arr],
             columns=["distribution", "value"]).to_csv("distributions.csv", index=False)

# --- 2. Real datasets used in this unit --------------------------------
sns.load_dataset("iris").to_csv("iris.csv", index=False)
sns.load_dataset("mpg").to_csv("mpg.csv", index=False)
sns.load_dataset("titanic")[["class", "sex", "age", "fare", "survived"]].to_csv("titanic.csv", index=False)
sns.load_dataset("diamonds")[["cut", "color", "price"]].to_csv("diamonds.csv", index=False)
sns.load_dataset("flights").to_csv("flights.csv", index=False)
px.data.gapminder().to_csv("gapminder.csv", index=False)

files_out = ["distributions.csv", "iris.csv", "mpg.csv", "titanic.csv",
             "diamonds.csv", "flights.csv", "gapminder.csv"]
for f in files_out:
    print(f"{f:<18}{len(pd.read_csv(f)):>7} rows")

# --- 3. Download them so you can upload them to the AI ------------------
try:
    from google.colab import files
    for f in files_out:
        files.download(f)
except ImportError:
    print("Not in Colab: the CSV files are in the current folder.")
```

```{note}
**Why upload files instead of letting the AI load the data?** The code sandboxes inside chat assistants usually have no internet access, so `sns.load_dataset(...)` fails there. Uploading the same CSV the Python cell used also guarantees that both sides start from identical data, which is what makes the comparison fair.
```

```{note}
**About the datasets.** The lecture slides use the Kaggle version of Gapminder (1998–2018, with HDI and CO₂), the Forest Fires and Mammographic Mass datasets, and Natural Earth geometries. This page uses datasets that load in Colab with one line and need no account: Plotly's Gapminder (142 countries, 1952–2007, every five years), and Seaborn's Iris, Auto MPG, Titanic, Diamonds, and Flights. The methods and the lessons are the same.
```

---

### 3.1 Distributions — Histogram, Boxplot, Violin

**Concept.** A distribution chart answers "what values does this variable take, and how often?"

| Chart | Use it to | Data | How to read it |
| :--- | :--- | :--- | :--- |
| **Histogram** | See the overall shape of one variable | Continuous | The range is divided into bins; bar height is the count in each bin. Look for center, spread, skewness, modes, gaps, and outliers. |
| **Boxplot** | Summarize one variable, or compare several, with the five-number summary (min, Q1, median, Q3, max) | Continuous | The box spans Q1 to Q3 (the IQR). Points beyond Q3 + γ·IQR or below Q1 − γ·IQR are drawn as outliers; most software uses γ = 1.5. |
| **Violin plot** | Combine a boxplot with a density curve | Continuous | Width is density: wide means many values, narrow means few. Shows shape that a boxplot cannot. |

The cell below draws five distributions with very different shapes — normal, uniform, Laplace (heavy-tailed), right-skewed, and bimodal — three ways each, then compares the sepal length of three Iris species.

```python
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
from scipy import stats

# Five distributions, 2,000 values each (seed 42, identical to Cell 0)
rng = np.random.default_rng(42)
n = 2000
dists = {
    "Normal":       rng.normal(0, 1, n),
    "Uniform":      rng.uniform(0, 1, n),
    "Laplace":      rng.laplace(0, 1, n),
    "Right-skewed": rng.beta(1.5, 8, n),
    "Bimodal":      np.concatenate([rng.normal(-2, 0.6, n // 2),
                                    rng.normal(2, 0.6, n // 2)]),
}

GAMMA = 1.5   # whisker rule: outlier if beyond Q1 - γ·IQR or Q3 + γ·IQR

print(f"{'distribution':<14}{'mean':>8}{'median':>8}{'skew':>7}"
      f"{'kurt*':>7}{'IQR':>7}{'outliers':>9}")
fig, axes = plt.subplots(5, 3, figsize=(13, 16))
for row, (name, x) in zip(axes, dists.items()):
    q1, q3 = np.percentile(x, [25, 75]); iqr = q3 - q1
    n_out = ((x < q1 - GAMMA * iqr) | (x > q3 + GAMMA * iqr)).sum()
    print(f"{name:<14}{x.mean():8.2f}{np.median(x):8.2f}{stats.skew(x):7.2f}"
          f"{stats.kurtosis(x):7.2f}{iqr:7.2f}{n_out:9d}")

    sns.histplot(x, bins=30, kde=True, ax=row[0])
    row[0].axvline(x.mean(), color="red", ls="--", label="mean")
    row[0].axvline(np.median(x), color="green", label="median")
    row[0].set_title(f"{name}: histogram"); row[0].legend()
    row[1].boxplot(x, vert=False, whis=GAMMA)
    row[1].set_title(f"{name}: boxplot ({n_out} 'outliers')")
    sns.violinplot(x=x, inner="quart", ax=row[2])
    row[2].set_title(f"{name}: violin")
plt.tight_layout(); plt.show()
print("* kurt = excess kurtosis (normal = 0). Some tools report plain kurtosis (normal = 3).")

# Comparing groups: Iris sepal length by species
iris = sns.load_dataset("iris")
fig, ax = plt.subplots(1, 2, figsize=(12, 5), sharey=True)
sns.boxplot(data=iris, x="species", y="sepal_length", hue="species", ax=ax[0])
sns.stripplot(data=iris, x="species", y="sepal_length", color="black", size=3, ax=ax[0])
ax[0].set_title("Boxplot with data points")
sns.violinplot(data=iris, x="species", y="sepal_length", hue="species", ax=ax[1])
sns.stripplot(data=iris, x="species", y="sepal_length", color="black", size=3, ax=ax[1])
ax[1].set_title("Violin plot with data points")
ax[0].set_ylabel("Sepal length (cm)")
plt.tight_layout(); plt.show()
print(iris.groupby("species")["sepal_length"].describe().round(2))
```

**What the Python cell prints** (seed 42):

| Distribution | Mean | Median | Skew | Excess kurtosis | Boxplot "outliers" |
| :--- | ---: | ---: | ---: | ---: | ---: |
| Normal | −0.06 | −0.02 | −0.05 | 0.00 | 17 |
| Uniform | 0.49 | 0.50 | 0.03 | −1.20 | 0 |
| Laplace | −0.05 | −0.03 | 0.10 | 3.21 | 122 |
| Right-skewed | 0.16 | 0.13 | 1.05 | 0.94 | 40 |
| Bimodal | −0.00 | −0.07 | −0.00 | −1.69 | 0 |

**Prompt** — upload `distributions.csv` and `iris.csv`, then paste:

```
I uploaded two files.

distributions.csv has 10,000 rows and two columns, "distribution" and "value". It holds five synthetic distributions (Normal, Uniform, Laplace, Right-skewed, Bimodal) with 2,000 values each.

iris.csv is the Iris flower dataset (150 flowers, three species, four measurements in centimeters).

Using Python, do the following and show me every chart:

1. Draw a 5 x 3 grid: one row per distribution, with a histogram plus density curve in the first column, a boxplot in the second, and a violin plot in the third. Mark the mean and the median on each histogram.
2. Print a table with the mean, median, skewness, kurtosis, interquartile range, and number of boxplot outliers for each distribution.
3. Draw the sepal length of the three Iris species as a boxplot and as a violin plot side by side, with the individual flowers shown as points on top.

Then, in no more than five sentences, explain what each chart type reveals that the other two hide. Show the code you used.
```

**Follow-up prompt** (same chat):

```
Look again at the Bimodal row. Where do its mean and median fall, and how many values actually lie near them? If a manager saw only the boxplot of this variable, what wrong conclusion could they draw?
```

**What to compare in this round:**

- **Kurtosis convention.** The Python cell reports *excess* kurtosis (normal = 0). Many tools report *plain* kurtosis (normal = 3). If the AI's values are about 3 higher, neither side is wrong — but the AI should have said which convention it used. Did it?
- **Outlier counts.** Did the AI use γ = 1.5? The 17 "outliers" in the Normal row and 122 in the Laplace row are not errors in the data; they are what the 1.5·IQR rule flags in a perfectly clean, heavy-tailed sample. Did the AI call them errors?
- **The bimodal case.** The boxplot shows a symmetric box with no outliers. Only the histogram and the violin show two separate groups, with the mean and median sitting in the empty gap between them. Did the AI notice?
- **Bins and smoothing.** Compare the number of histogram bins and the smoothness of the density curves. Small changes here change what shape you think you see.

**Try changing it:**

1. Set `GAMMA = 3.0` and run again. How many Normal and Laplace "outliers" survive? What does this tell you about outliers detected by a rule rather than by knowledge of the data?
2. In the violin line, add `cut=0` to get `sns.violinplot(x=x, inner="quart", cut=0, ax=row[2])`. Look at the Uniform violin before and after: the default version draws values below 0 and above 1 that cannot exist.
3. Change `bins=30` to `bins=4`, then to `bins=100`. With 4 bins, is there any sign left that the Bimodal data has two separate groups?

---

### 3.2 Associations — Scatter Plot, Bubble Chart, Scatterplot Matrix, Heatmap

**Concept.** An association chart answers "when one variable changes, what does another one do?"

| Chart | Use it to | Data | How to read it |
| :--- | :--- | :--- | :--- |
| **Scatter plot** | See the relationship between two variables | Continuous on both axes; a category can go in color | Look at direction, strength, and form (straight or curved), plus clusters and outliers. Pearson's correlation (PCC) measures only the *straight-line* part. |
| **Bubble chart** | Add a third and fourth variable to a scatter plot | Continuous axes and size; categorical color | Bubble size encodes a variable in *area*, a channel people read poorly. Use it for the big picture, not for precise values. |
| **Scatterplot matrix** | See every pair of variables at once | Several continuous variables | Off-diagonal panels are pairwise scatter plots; diagonal panels show each variable's distribution. |
| **Heatmap / correlogram** | Show a matrix of values, usually correlations, as colors | Numerical | Color intensity encodes magnitude. The color scale you choose decides what looks "strong." |

```python
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import plotly.express as px

mpg = sns.load_dataset("mpg").dropna()          # drops 6 cars with missing horsepower
print(f"Cars used: {len(mpg)} of 398")

# --- 1. Scatter plots with Pearson correlation (PCC) -------------------
pairs = [("weight", "mpg"), ("horsepower", "mpg"),
         ("acceleration", "mpg"), ("horsepower", "acceleration")]
fig, axes = plt.subplots(2, 2, figsize=(11, 8))
for ax, (x, y) in zip(axes.flat, pairs):
    r = mpg[x].corr(mpg[y])
    sns.regplot(data=mpg, x=x, y=y, ax=ax, scatter_kws={"s": 15, "alpha": .6})
    ax.set_title(f"{y} vs {x}   PCC = {r:.2f}")
    print(f"PCC({x}, {y}) = {r:.2f}")
plt.tight_layout(); plt.show()

# --- 2. Bubble chart: four variables in one picture --------------------
fig, ax = plt.subplots(figsize=(10, 6))
sns.scatterplot(data=mpg, x="acceleration", y="mpg", hue="cylinders",
                size="horsepower", sizes=(15, 300), alpha=.6, palette="viridis", ax=ax)
ax.set_title("MPG vs acceleration (colour = cylinders, size = horsepower)")
ax.legend(bbox_to_anchor=(1.02, 1), loc="upper left")
plt.tight_layout(); plt.show()

gap07 = px.data.gapminder().query("year == 2007")
fig = px.scatter(gap07, x="gdpPercap", y="lifeExp", size="pop", color="continent",
                 hover_name="country", log_x=True, size_max=60,
                 labels={"gdpPercap": "GDP per capita (US$, log scale)",
                         "lifeExp": "Life expectancy (years)"},
                 title="Life expectancy vs GDP per capita, 2007 (size = population)")
fig.show()

# --- 3. Scatterplot matrix ---------------------------------------------
iris = sns.load_dataset("iris")
sns.pairplot(iris, hue="species", height=2.2)
plt.show()

# --- 4. Correlation heatmap --------------------------------------------
num = mpg.select_dtypes("number")
corr = num.corr().round(2)
plt.figure(figsize=(8, 6))
sns.heatmap(corr, annot=True, cmap="coolwarm", vmin=-1, vmax=1, center=0)
plt.title("Auto MPG: correlation heatmap (scale fixed at -1 to +1)")
plt.tight_layout(); plt.show()
```

**What the Python cell prints:** 392 of 398 cars are used (6 have no horsepower value). PCC(weight, mpg) = −0.83; PCC(horsepower, mpg) = −0.78; PCC(acceleration, mpg) = 0.42; PCC(horsepower, acceleration) = −0.69.

**Prompt** — upload `mpg.csv`, `gapminder.csv`, and `iris.csv`, then paste:

```
I uploaded three files: mpg.csv (Auto MPG, 398 cars from 1970 to 1982), gapminder.csv (142 countries, 1952 to 2007), and iris.csv (the Iris flower dataset).

Using Python, do the following and show me every chart:

1. From mpg.csv, draw four scatter plots in a 2 x 2 grid, each with a fitted regression line and the Pearson correlation in its title: mpg vs weight, mpg vs horsepower, mpg vs acceleration, and acceleration vs horsepower.
2. From mpg.csv, draw a bubble chart of mpg vs acceleration, with color showing the number of cylinders and bubble size showing horsepower.
3. From gapminder.csv, using only the year 2007, draw a bubble chart of life expectancy vs GDP per capita, with bubble size showing population and color showing continent.
4. From iris.csv, draw a scatterplot matrix of the four measurements, colored by species.
5. From mpg.csv, draw a correlation heatmap of all numeric columns, with the values written in each cell.

Tell me how many rows you used from mpg.csv and why. Then, in no more than five sentences, describe the strongest relationships you see. Show the code you used.
```

**Follow-up prompt** (same chat):

```
Look again at the scatter plot of mpg vs horsepower. Is the relationship a straight line or a curve? Given your answer, is a Pearson correlation of about -0.78 a fair one-number summary of it? What would you report instead?
```

**What to compare in this round:**

- **Rows used.** `horsepower` has 6 missing values. Did the AI drop them, fill them in, or fail silently? Did it tell you?
- **GDP axis.** The Python bubble chart uses a **log scale** for GDP per capita. If the AI used a linear scale, most of the world's countries are squeezed against the left edge. Same data, different story.
- **Heatmap color scale.** The Python heatmap fixes the scale at −1 to +1, centered on 0, so that white means "no correlation." If the AI let the scale fit the data, check what color 0.29 or −0.31 turned into. Does a weak correlation now look strong?
- **Correlation is not the whole relationship.** In the mpg vs horsepower panel, the straight regression line runs under the data at both ends. The relationship is curved. Did the AI's interpretation say so?

**Try changing it:**

1. In the heatmap line, delete `vmin=-1, vmax=1, center=0` and run again. Which correlations look stronger than before, even though no number changed?
2. In the Gapminder bubble chart, change `log_x=True` to `log_x=False`. Which continent's story disappears?
3. In the `pairs` list, replace `("acceleration", "mpg")` with `("model_year", "mpg")`. Cars got more efficient over the decade — but is the year the *cause*? Which other variable in the heatmap moves with the year?

---

### 3.3 Amounts — Bar Chart and Radar Chart

**Concept.** An amount chart answers "how much, for each category?"

| Chart | Use it to | Data | How to read it |
| :--- | :--- | :--- | :--- |
| **Bar chart** | Compare magnitudes across discrete categories, or across two categorical variables at once (grouped bars) | Categorical or discrete | Bar length encodes the value, so the axis **must start at zero**. Grouped bars let you compare within and across groups. |
| **Radar chart** | Compare several items across many variables at once | Continuous, ideally on a common scale | Each item is a polygon; values farther from the center are larger. Polygon shape and area depend on the **order of the axes**, which is arbitrary. |

```python
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

titanic = sns.load_dataset("titanic")
titanic["outcome"] = titanic["survived"].map({0: "Died", 1: "Survived"})
order = ["First", "Second", "Third"]

# --- 1. Three bar charts of the same data ------------------------------
counts = pd.crosstab(titanic["class"], titanic["outcome"]).reindex(order)
print(counts, "\n")
print("Survival rate by class:")
print((counts["Survived"] / counts.sum(axis=1)).round(2), "\n")

fig, ax = plt.subplots(1, 3, figsize=(16, 5))
titanic["class"].value_counts().reindex(order).plot.bar(ax=ax[0], color="steelblue")
ax[0].set_title("A. Passengers per class"); ax[0].set_ylabel("Passengers")

combo = counts.stack(); combo.index = [f"{c}, {o}" for c, o in combo.index]
combo.plot.bar(ax=ax[1], color="gray")
ax[1].set_title("B. Class & outcome as one label"); ax[1].set_ylabel("Passengers")

counts.plot.bar(ax=ax[2], color=["firebrick", "seagreen"])
ax[2].set_title("C. Grouped bars: class vs outcome"); ax[2].set_ylabel("Passengers")
for a in ax: a.tick_params(axis="x", rotation=45)
plt.tight_layout(); plt.show()

# --- 2. Radar chart: profiles across many variables --------------------
def radar(ax, labels, series, title, rmax=None):
    angles = np.linspace(0, 2 * np.pi, len(labels), endpoint=False).tolist()
    angles += angles[:1]
    for name, values in series.items():
        v = list(values) + [values[0]]
        ax.plot(angles, v, label=name)
        ax.fill(angles, v, alpha=.15)
    ax.set_xticks(angles[:-1]); ax.set_xticklabels(labels, fontsize=8)
    if rmax: ax.set_ylim(0, rmax)
    ax.set_title(title, pad=18)
    ax.legend(loc="upper center", bbox_to_anchor=(0.5, -0.08), ncol=3, fontsize=8)

iris = sns.load_dataset("iris")
means = iris.groupby("species").mean()
print("Iris means (cm):"); print(means.round(2), "\n")

careers = pd.DataFrame({
    "Business Analyst": [40, 50, 55, 95, 80, 55, 25, 20],
    "Data Analyst":     [40, 55, 100, 55, 100, 95, 40, 40],
    "ML Engineer":      [100, 100, 100, 55, 55, 75, 85, 85],
    "Data Scientist":   [80, 100, 100, 70, 55, 60, 70, 75],
    "Data Engineer":    [80, 55, 55, 50, 55, 55, 100, 80],
}, index=["Programming", "Predictive stats", "Descriptive stats", "Business analysis",
          "Storytelling", "Visualization", "Databases", "Software eng."])

fig = plt.figure(figsize=(13, 6))
radar(fig.add_subplot(1, 2, 1, polar=True), list(means.columns),
      {s: means.loc[s].values for s in means.index}, "Iris: mean measurements (cm)")
radar(fig.add_subplot(1, 2, 2, polar=True), list(careers.index),
      {c: careers[c].values for c in ["Data Analyst", "ML Engineer", "Data Engineer"]},
      "Illustrative skill profiles (0-100)", rmax=100)
plt.tight_layout(); plt.show()
```

**What the Python cell prints:** First class 136 survived / 80 died; Second 87 / 97; Third 119 / 372. Survival rate: First 0.63, Second 0.47, Third 0.24.

**Prompt** — upload `titanic.csv` and `iris.csv`, then paste:

```
I uploaded titanic.csv (891 passengers; columns class, sex, age, fare, survived, where survived = 1 means the passenger survived) and iris.csv (the Iris flower dataset).

Using Python, do the following and show me every chart:

1. From titanic.csv, draw three bar charts side by side: (A) the number of passengers in each class; (B) one bar for every combination of class and outcome (survived or died), with the combination as the bar label; (C) a grouped bar chart with class on the x-axis and one bar for "survived" and one for "died" in each class. Print the counts behind chart C and the survival rate of each class.
2. From iris.csv, draw a radar chart with one polygon per species showing the mean of the four measurements.
3. Draw a second radar chart from this table of illustrative skill levels (0 to 100), using only Data Analyst, ML Engineer, and Data Engineer:

Skill, Business Analyst, Data Analyst, ML Engineer, Data Scientist, Data Engineer
Programming, 40, 40, 100, 80, 80
Predictive stats, 50, 55, 100, 100, 55
Descriptive stats, 55, 100, 100, 100, 55
Business analysis, 95, 55, 55, 70, 50
Storytelling, 80, 100, 55, 55, 55
Visualization, 55, 95, 75, 60, 55
Databases, 25, 40, 85, 70, 100
Software eng., 20, 40, 85, 75, 80

Then tell me which of the three Titanic bar charts answers the question "did class affect survival?" most directly, and why. Show the code you used.
```

**Follow-up prompt** (same chat):

```
Redraw the skill radar chart with the eight skills in this order instead: Programming, Storytelling, Databases, Visualization, Software eng., Business analysis, Predictive stats, Descriptive stats. Put the two versions side by side. The data did not change. What changed in how the three roles look, and what does that tell you about reading shapes and areas in a radar chart?
```

**What to compare in this round:**

- **Category order.** The Python cell orders classes First, Second, Third. Did the AI sort them alphabetically, by count, or in the natural order? An alphabetical "First, Second, Third" happens to work here; it would not for "Low, Medium, High."
- **Chart B vs chart C.** Both show the same six numbers. In which one can you see the class effect on survival at a glance? This is the difference between *displaying* data and *answering a question*.
- **Radar scale.** The Python skill radar fixes the radius at 0–100. If the AI let the radius start above 0, the smaller values shrink toward the center and differences look larger.
- **Iris radar.** All four Iris measurements are in centimeters, so a shared radius is fair. Ask yourself whether it would still be fair if one variable were in millimeters.

**Try changing it:**

1. Change `["Data Analyst", "ML Engineer", "Data Engineer"]` to `["Business Analyst", "Data Scientist", "Data Engineer"]`. Which pair of roles is now hardest to tell apart, and is that because of the data or because of the overlapping fills?
2. Right after the `careers` table, add the line `careers = careers.iloc[[0, 4, 6, 5, 7, 3, 1, 2]]`. This only reorders the axes (the same order as the follow-up prompt). Which role's polygon changes most, and did any value change?
3. In chart C, change `counts.plot.bar(...)` to `counts.div(counts.sum(axis=1), axis=0).plot.bar(...)`. You now plot survival *rates* instead of counts. Which class looks different, and which version would you show a historian?

---

### 3.4 Proportions — Pie, Doughnut, Treemap, Sunburst

**Concept.** A proportion chart answers "what share of the whole does each part represent?"

| Chart | Use it to | Data | How to read it |
| :--- | :--- | :--- | :--- |
| **Pie chart** | Show the share of a few mutually exclusive categories | Categorical | Slice angle and area encode share. Readable for two or three very different slices; unreadable for many similar ones. |
| **Doughnut chart** | A pie with the center removed | Categorical | Encodes share in arc length. Same strengths and weaknesses as the pie. |
| **Treemap** | Show hierarchical shares (for example, continent → country) | Hierarchical categories plus a size value | Rectangle area is the share; color can encode a second variable. |
| **Sunburst** | Show the same hierarchy as rings | Hierarchical | The inner ring is the top level; each outer ring splits it further. |

The cell puts a pie and a bar chart of the **same numbers** side by side — once for 5 categories with clearly different shares, once for 7 categories with similar shares.

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import plotly.express as px

diamonds = sns.load_dataset("diamonds")
cut = diamonds["cut"].value_counts(normalize=True).mul(100).round(1)
color = diamonds["color"].value_counts(normalize=True).mul(100).round(1)
print("Cut (%):\n", cut.to_string(), "\n\nColour grade (%):\n", color.to_string())

# --- 1. Pie vs bar, same numbers ---------------------------------------
fig, ax = plt.subplots(2, 2, figsize=(13, 10))
for row, (name, s) in enumerate([("Cut (5 categories)", cut),
                                 ("Colour grade (7 categories)", color)]):
    ax[row, 0].pie(s, labels=s.index, autopct="%1.1f%%", startangle=90,
                   counterclock=False)
    ax[row, 0].set_title(f"Pie: {name}")
    s.plot.bar(ax=ax[row, 1], color="steelblue")
    ax[row, 1].set_title(f"Bar: {name}"); ax[row, 1].set_ylabel("% of diamonds")
    ax[row, 1].tick_params(axis="x", rotation=0)
plt.tight_layout(); plt.show()

# --- 2. Doughnut --------------------------------------------------------
fig, ax = plt.subplots(figsize=(6, 6))
ax.pie(cut, labels=cut.index, autopct="%1.1f%%", startangle=90,
       counterclock=False, pctdistance=0.8, wedgeprops={"width": 0.4})
ax.set_title("Doughnut: diamond cut")
plt.show()

# --- 3. Treemap and sunburst: hierarchical proportions -----------------
gap07 = px.data.gapminder().query("year == 2007")
print(f"\nWorld population in the data (2007): {gap07['pop'].sum()/1e9:.2f} billion")
print((gap07.groupby("continent")["pop"].sum() / gap07["pop"].sum() * 100)
      .round(1).sort_values(ascending=False).to_string())

fig = px.treemap(gap07, path=[px.Constant("World"), "continent", "country"],
                 values="pop", color="lifeExp", color_continuous_scale="RdYlGn",
                 title="Treemap: population 2007 (area) and life expectancy (colour)")
fig.show()

fig = px.sunburst(gap07, path=["continent", "country"], values="pop",
                  title="Sunburst: the same hierarchy as rings")
fig.show()
```

**What the Python cell prints:** Cut — Ideal 40.0%, Premium 25.6%, Very Good 22.4%, Good 9.1%, Fair 3.0%. Colour grade — G 20.9%, E 18.2%, F 17.7%, H 15.4%, D 12.6%, I 10.1%, J 5.2%. Population 2007 in this dataset: 6.25 billion; Asia 61.0%, Africa 14.9%, Americas 14.4%, Europe 9.4%, Oceania 0.4%.

**Prompt** — upload `diamonds.csv` and `gapminder.csv`, then paste:

```
I uploaded diamonds.csv (53,940 diamonds; columns cut, color, price) and gapminder.csv (142 countries, 1952 to 2007; columns include country, continent, year, lifeExp, pop).

Using Python, do the following and show me every chart:

1. Compute the percentage of diamonds in each cut and in each color grade, and print both tables.
2. Draw a 2 x 2 grid: a pie chart and a bar chart of the cut percentages in the first row, and a pie chart and a bar chart of the color grade percentages in the second row. Put the percentage on every pie slice.
3. Draw a doughnut chart of the cut percentages.
4. Using only the year 2007 from gapminder.csv, draw a treemap of population with the hierarchy World, then continent, then country, and color each country by life expectancy.
5. Draw a sunburst chart of the same 2007 population hierarchy (continent, then country).

Then answer: in the color grade pie chart, can you tell without reading the labels whether E or F is larger? Show the code you used.
```

**Follow-up prompt** (same chat):

```
Diamond color grades are ordered from D (best, colorless) to J (most tinted). Redraw the color grade bar chart in that order, D to J. Compared with the chart sorted by frequency, which version answers the question "do better color grades become rarer?" and which version answers "which grade is most common?"
```

**What to compare in this round:**

- **Readability test.** E (18.2%) and F (17.7%) differ by half a percentage point. In the pie, they are indistinguishable; in the bar chart, you can see the difference. This is the Part II warning about pie charts, demonstrated on real data.
- **Slice order.** Did the AI start the pie at 12 o'clock and go clockwise, sorted by size? Matplotlib's default starts at 3 o'clock and goes counterclockwise. Neither is wrong, but one is far easier to read.
- **Ordinal categories.** The CSV file does not remember that cut and color are ordered. Did the AI sort cut as Ideal → Fair, alphabetically, or by count?
- **Treemap tool.** Did the AI use an interactive tool (Plotly) or a static one? If it could not draw a treemap, what did it draw instead — and did it tell you?

**Try changing it:**

1. Add the line `color = color.reindex(list("DEFGHIJ"))` right after `color` is computed. Which story does the bar chart tell now that it did not tell before?
2. In the treemap, change `values="pop"` to `values="gdpPercap"`. The rectangle sizes now add up GDP *per person* across countries. Is that sum meaningful? What should you plot instead?
3. In the doughnut, change `"width": 0.4` to `"width": 0.1`, then to `"width": 1.0`. At which width does it become a pie, and at which does it become hard to read?

---

### 3.5 Evolution and Flow — Line Chart, Sankey Diagram, Gantt Chart

**Concept.** These charts answer "how does something change over time, or move between states?"

| Chart | Use it to | Data | How to read it |
| :--- | :--- | :--- | :--- |
| **Line chart** | Show trends, cycles, spikes, and dips over time or another ordered variable | A continuous sequence | Look for the overall trend, changes in direction, repeating cycles, and anomalies. |
| **Sankey diagram** | Show quantities flowing through stages of a process | Categorical stages with a numeric flow size | Nodes are stages; link width is the amount moving between them. Wider means more. |
| **Gantt chart** | Show tasks or stages on a timeline | Named tasks with start and end dates | Each bar starts at the start date and ends at the end date; length is duration. Used mainly in project management. |

```python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import plotly.express as px
import plotly.graph_objects as go

# --- 1. Line charts: trend and cycle -----------------------------------
flights = sns.load_dataset("flights")
fig, ax = plt.subplots(1, 2, figsize=(14, 5))
yearly = flights.groupby("year")["passengers"].sum()   # already in thousands
ax[0].plot(yearly.index, yearly.values, "o-")
ax[0].set_title("Trend: total airline passengers per year")
ax[0].set_xlabel("Year"); ax[0].set_ylabel("Passengers (thousands)"); ax[0].set_ylim(0)
sns.lineplot(data=flights, x="month", y="passengers", hue="year",
             palette="viridis", ax=ax[1])
ax[1].set_title("Cycle: monthly passengers, one line per year")
ax[1].set_ylabel("Passengers (thousands)")
ax[1].legend(ncol=2, fontsize=7)
plt.tight_layout(); plt.show()
print(f"Passengers grew from {yearly.iloc[0]:,} thousand (1949) to {yearly.iloc[-1]:,} thousand (1960)")
print("Busiest month every year:",
      flights.loc[flights.groupby("year")["passengers"].idxmax(), "month"].unique().tolist())

# --- 2. Line chart by group: two ways to average ------------------------
gap = px.data.gapminder()
simple = gap.groupby(["continent", "year"])["lifeExp"].mean().unstack(0)
gap["w"] = gap["lifeExp"] * gap["pop"]
g = gap.groupby(["continent", "year"])[["w", "pop"]].sum()
weighted = (g["w"] / g["pop"]).unstack(0)

fig, ax = plt.subplots(1, 2, figsize=(14, 5), sharey=True)
simple.plot(ax=ax[0], marker="o", ms=3)
ax[0].set_title("Average of countries (every country counts once)")
weighted.plot(ax=ax[1], marker="o", ms=3)
ax[1].set_title("Average of people (weighted by population)")
ax[0].set_ylabel("Life expectancy (years)")
plt.tight_layout(); plt.show()
print("Asia, 1962:  country average = %.1f   population-weighted = %.1f"
      % (simple.loc[1962, "Asia"], weighted.loc[1962, "Asia"]))

# --- 3. Sankey: how people flow between stages (illustrative data) -----
stages = ["Undergraduate", "M.Sc.", "Ph.D.", "Postdoc", "Faculty", "Industry"]
flows = [  # (from, to, people)
    ("Undergraduate", "Industry", 600), ("Undergraduate", "M.Sc.", 300),
    ("Undergraduate", "Ph.D.", 100),    ("M.Sc.", "Industry", 180),
    ("M.Sc.", "Ph.D.", 120),            ("Ph.D.", "Postdoc", 120),
    ("Ph.D.", "Industry", 70),          ("Ph.D.", "Faculty", 30),
    ("Postdoc", "Faculty", 50),         ("Postdoc", "Industry", 70),
]
idx = {s: i for i, s in enumerate(stages)}
fig = go.Figure(go.Sankey(
    node=dict(label=stages, pad=20),
    link=dict(source=[idx[a] for a, b, v in flows],
              target=[idx[b] for a, b, v in flows],
              value=[v for a, b, v in flows])))
fig.update_layout(title="Where 1,000 graduates end up (illustrative numbers)")
fig.show()
for stage in ["Industry", "Faculty"]:
    print(f"End in {stage}: {sum(v for a, b, v in flows if b == stage)}")

# --- 4. Gantt: one career as a timeline (illustrative dates) -----------
career = pd.DataFrame([
    ("Undergraduate", "2022-08-15", "2026-05-01"),
    ("M.Sc.",         "2026-08-15", "2028-05-01"),
    ("Ph.D.",         "2028-08-15", "2032-05-01"),
    ("Postdoc",       "2032-06-01", "2034-06-01"),
    ("Faculty",       "2034-08-15", "2044-05-01"),
    ("Industry",      "2044-06-01", "2055-12-31"),
], columns=["Stage", "Start", "End"])
fig = px.timeline(career, x_start="Start", x_end="End", y="Stage",
                  title="Gantt: one person's career (illustrative)")
fig.update_yaxes(autorange="reversed")
fig.show()
```

**What the Python cell prints:** Passengers grew from 1,520 thousand (1949) to 5,714 thousand (1960); the busiest month is always July or August. Asia, 1962: country average = 51.6 years, population-weighted average = 46.6 years. Sankey: 920 of 1,000 people end in Industry, 80 in Faculty.

```{note}
The Sankey and Gantt data are **illustrative**, invented to show how the charts work. They are not career statistics. This matters: a Sankey diagram looks equally authoritative whether its numbers come from a census or from a guess.
```

**Prompt** — upload `flights.csv` and `gapminder.csv`, then paste:

```
I uploaded flights.csv (monthly airline passengers, 1949 to 1960, in thousands; columns year, month, passengers) and gapminder.csv (142 countries, 1952 to 2007; columns include country, continent, year, lifeExp, pop).

Using Python, do the following and show me every chart:

1. From flights.csv, draw two line charts side by side: total passengers per year (the trend), and passengers per month with one line per year (the cycle). Tell me the busiest month.
2. From gapminder.csv, draw the life expectancy of each continent over time as a line chart. Print the value you computed for Asia in 1962.
3. Draw a Sankey diagram from this illustrative table of where 1,000 graduates end up:

From, To, People
Undergraduate, Industry, 600
Undergraduate, M.Sc., 300
Undergraduate, Ph.D., 100
M.Sc., Industry, 180
M.Sc., Ph.D., 120
Ph.D., Postdoc, 120
Ph.D., Industry, 70
Ph.D., Faculty, 30
Postdoc, Faculty, 50
Postdoc, Industry, 70

4. Draw a Gantt chart of this illustrative career:

Stage, Start, End
Undergraduate, 2022-08-15, 2026-05-01
M.Sc., 2026-08-15, 2028-05-01
Ph.D., 2028-08-15, 2032-05-01
Postdoc, 2032-06-01, 2034-06-01
Faculty, 2034-08-15, 2044-05-01
Industry, 2044-06-01, 2055-12-31

Then, in no more than five sentences, describe the main pattern in each chart. Show the code you used.
```

**Follow-up prompt** (same chat):

```
For the continent life expectancy chart, did you average the countries (each country counts once) or average the people (weight each country by its population)? Draw both versions side by side and tell me the Asia value for 1962 in each. Why are they different, and which one answers the question "how long did a typical person in Asia expect to live?"
```

**What to compare in this round:**

- **The average hidden inside the line.** "Life expectancy of each continent" sounds like one thing, but it can be computed two ways. The Python cell draws both: in 1962, Asia is 51.6 years averaged over countries and 46.6 years averaged over people. The population-weighted line shows a sharp dip around 1962 — the famine in China — that the country average hides. Which one did the AI compute without being asked?
- **Units.** `passengers` is already in thousands. Did the AI label the axis correctly, or did it say "passengers" for numbers that are thousands of passengers?
- **Trend axis.** The Python trend chart starts the y-axis at 0. Check the AI's version against the Part II hands-on below: does the visual growth match the real growth?
- **Sankey layout.** Plotly places Sankey nodes automatically. Did the AI's diagram keep the stages in a sensible left-to-right order?

**Try changing it:**

1. In the `flows` list, change `("Ph.D.", "Postdoc", 120)` to `("Ph.D.", "Postdoc", 300)`. More people now leave the Ph.D. node than entered it. Does Plotly warn you? What does that tell you about checking the numbers behind a polished chart?
2. In the trend chart, delete `ax[0].set_ylim(0)`. By how much does the growth appear to change?
3. In the Gantt table, move the `Postdoc` start date to `"2031-06-01"`, so that it overlaps the Ph.D. Is an overlap a data error, or could it be real? What would you need to know?

---

### 3.6 Geospatial — Choropleth Map and Bubble Map

**Concept.** A geospatial chart answers "how does something vary across places?"

| Chart | Use it to | Data | How to read it |
| :--- | :--- | :--- | :--- |
| **Choropleth map** | Color each region by a value | One value per region (country, state, county) | Darker usually means higher. **Unclassed** maps use a continuous color scale; **classed** maps group values into bins. Large regions dominate the eye regardless of their value. |
| **Bubble map** | Place a sized circle on each location | One value per location | Bubble size encodes the value, so small regions with large values stay visible. |

```python
import pandas as pd
import plotly.express as px

gap = px.data.gapminder()
gap07 = gap.query("year == 2007").copy()

# --- 1. Choropleth, unclassed: a continuous colour scale ---------------
fig = px.choropleth(gap07, locations="iso_alpha", color="pop",
                    hover_name="country", color_continuous_scale="Reds",
                    title="Population 2007, unclassed (continuous scale)")
fig.show()

# --- 2. Choropleth, classed: the same data in bins ---------------------
bins = [0, 5e6, 20e6, 50e6, 100e6, 250e6, 2e9]
labels = ["< 5M", "5-20M", "20-50M", "50-100M", "100-250M", "> 250M"]
gap07["pop_class"] = pd.cut(gap07["pop"], bins=bins, labels=labels)
fig = px.choropleth(gap07, locations="iso_alpha", color="pop_class",
                    hover_name="country", category_orders={"pop_class": labels},
                    color_discrete_sequence=px.colors.sequential.Reds[2:],
                    title="Population 2007, classed (6 bins)")
fig.show()
print(gap07["pop_class"].value_counts().reindex(labels).to_string())
share = gap07.nlargest(2, "pop")["pop"].sum() / gap07["pop"].sum() * 100
print(f"\nChina + India = {share:.0f}% of the population in this dataset")

# --- 3. Animated choropleth: a rate, not a total -----------------------
fig = px.choropleth(gap, locations="iso_alpha", color="lifeExp",
                    hover_name="country", animation_frame="year",
                    range_color=[25, 85], color_continuous_scale="Viridis",
                    title="Life expectancy (years), 1952-2007: press play")
fig.show()

# --- 4. Bubble map: size on top of geography ---------------------------
gap07["gdp_billion"] = gap07["pop"] * gap07["gdpPercap"] / 1e9
fig = px.scatter_geo(gap07, locations="iso_alpha", size="gdp_billion",
                     color="continent", hover_name="country", size_max=50,
                     projection="natural earth",
                     title="Bubble map: total GDP 2007 (US$ billions)")
fig.show()
print(gap07.nlargest(5, "gdp_billion")[["country", "gdp_billion"]]
      .round(0).to_string(index=False))
```

**What the Python cell prints:** Countries per population class — < 5M: 40; 5–20M: 52; 20–50M: 28; 50–100M: 12; 100–250M: 7; > 250M: 3. China and India together are 39% of the population in the dataset. Largest total GDP in 2007: United States, China, Japan, India, Germany.

```{note}
Plotly draws country borders by downloading them in your browser, so these maps need an internet connection. Colab has one. The code sandbox of a chat assistant usually does not, which leads directly to the first comparison point below.
```

**Prompt** — upload `gapminder.csv`, then paste:

```
I uploaded gapminder.csv (142 countries, 1952 to 2007; columns country, continent, year, lifeExp, pop, gdpPercap, iso_alpha, where iso_alpha is the three-letter country code).

Using Python, do the following and show me every map:

1. Using only 2007, draw a world choropleth map of population with a continuous color scale.
2. Draw the same map again with population grouped into these six classes: under 5 million, 5 to 20 million, 20 to 50 million, 50 to 100 million, 100 to 250 million, over 250 million. Print how many countries fall in each class.
3. Draw a world choropleth map of life expectancy that can be animated over the years 1952 to 2007, with the color scale fixed from 25 to 85 years.
4. Using only 2007, compute each country's total GDP as population times GDP per capita, and draw a world bubble map with bubble size showing total GDP and color showing continent. Print the five countries with the largest total GDP.

If you cannot draw world maps in your environment, tell me exactly why, and give me code I can run in Google Colab instead. Then, in no more than five sentences, compare what the unclassed and classed population maps make you see. Show the code you used.
```

**Follow-up prompt** (same chat):

```
In the population choropleth, Russia and Canada are among the largest shapes on the map. Does their size on the map tell me anything about their population? Explain the difference between mapping a total (like population) and mapping a rate (like life expectancy or population density), and tell me which of the four maps I asked for are maps of totals.
```

**What to compare in this round:**

- **Could the AI draw the map at all?** If its sandbox has no internet, it may not have world borders. Record what it did: drew the map anyway, drew a bar chart instead, plotted dots by coordinates, or told you it could not. Only the last one is honest by default. Then run the Colab code it gave you.
- **Unclassed vs classed.** In the unclassed map, nearly everything is pale except China and India, because two countries hold 39% of the population. The classed map shows differences among the other 140 countries. Same data, different message — and the bin edges are a choice someone made.
- **Totals vs rates.** A population map is mostly a map of where the large countries are. Life expectancy is a rate, so it can be compared across countries of any size.
- **Bubble map.** The United States has the largest bubble (about 12.9 trillion US$). Compare: is it clearer on the bubble map or on a choropleth of total GDP?

**Try changing it:**

1. Merge the two top classes: change `bins` to `[0, 5e6, 20e6, 50e6, 100e6, 2e9]` and `labels` to `["< 5M", "5-20M", "20-50M", "50-100M", "> 100M"]`. How many countries are now in the top class, and does the map look calmer or more alarming?
2. In the first map, change `color="pop"` to `color="lifeExp"`. Does Russia's large area still dominate your attention?
3. In the bubble map, change `projection="natural earth"` to `projection="mercator"`. Which regions grow, and does that change your impression of the data?

```{note}
**Starred methods in the slides.** Word clouds and dendrograms appear in the lecture slides as optional methods and are not part of the in-class rounds. The sunburst is included in 3.4 because it is one line of code once the treemap exists.
```

---

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

## Part IV — Using AI for Visualization

AI is strong at chart **implementation** and weak at chart **judgment**, which mirrors every other unit in Part IV and V. The six rounds in Part III should have shown you both halves: the AI produced working charts quickly, and it also chose a kurtosis convention, an averaging method, a color scale, and a sort order without telling you.

**Effective prompts:**

```
Write matplotlib code for the following.

Question the chart must answer: [do resolution times differ across severity levels?]
Data: <~800 rows; "severity" is one of four ordered categories; "minutes" is heavily right-skewed with outliers I want visible, not clipped>

Rules: label both axes with units. Do not truncate the y-axis. Do not add a title that merely restates the axis labels. Return only code.
```

```
Below is the code for a chart I built. I am claiming [your claim] from it.

You are a skeptical reviewer. What could you legitimately object to about how this is drawn? Check specifically: axis truncation, missing baseline, unlabeled units, a selected date range, aggregation that hides variance, and colour choices that imply a judgment.

For each objection, say how I would fix it and what I would lose by fixing it.

CODE: [PASTE]
```

```
I want to answer [your question] from data with columns [list them].

Suggest three different chart types. For each: what it would reveal, what it would hide, and which audience it suits.

Do not recommend one. I will choose.
```

**Ineffective prompts:**

> *"Make a nice visualization of this data."*

You get a default chart — usually a bar chart, usually with a title restating the axis labels. Defaults are not answers to questions.

```{note}
The second prompt above — asking for objections to your own chart — is the most useful of the three. It reliably surfaces truncated axes, missing baselines, unlabeled units, and cherry-picked ranges, and it does so without the emotional investment you have in the result. It is the visualization equivalent of the adversarial reading from Unit 13.
```

## Part V — From Chart to Story

A dashboard full of accurate charts can still fail, because the reader does not know where to look or what to do.

**Three questions before publishing any chart or dashboard:**

1. **Who reads this, and what decision do they make?** A chart for an engineer debugging a regression and a chart for a director allocating budget are different charts, even from identical data.
2. **What is the one thing they should take away?** If you cannot say it in a sentence, the chart is not finished.
3. **What would change their mind?** A visualization that cannot be contradicted by the data is decoration.

For dashboards specifically: order by decision priority, not by data availability. Most inherited dashboards are ordered by what was easy to query, which is why nobody looks at them.

## 💡 Example: The Chart That Ended the Argument

A team argued for months about whether their API was getting slower. Both sides had charts. One showed mean latency, flat. The other showed p99 latency, rising sharply.

Both were correct. The mean was flat because most requests were unaffected. The p99 was rising because a growing subset — the users with the largest accounts, and the ones most likely to churn — were experiencing severe degradation.

The chart that ended the argument was a distribution plot per month, showing a bimodal shape emerging over time. No summary statistic showed it. Anscombe's lesson, in production — and the Bimodal row of Section 3.1, in production.

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
- de Castro, L. N. _AEDA — Data Visualization Methods_ (lecture slides).
- Plotly Express documentation: [plotly.com/python/plotly-express](https://plotly.com/python/plotly-express/).
