# Unit 6: AI Applications Across Domains

> **Sessions 16–17** · Oct 12, Oct 14 · **HW6 assigned Oct 14, due Oct 21**

You now know how AI systems are built and where they break. This unit surveys where they are actually deployed — and, more usefully, teaches you to ask the same five questions of any application claim you encounter.

## Learning Objectives

After completing this unit, you will be able to:

- Survey **major AI applications** across healthcare, education, law, business, finance, and creative work.
- Apply a consistent **evaluation framework** to any claimed AI application.
- Distinguish **decision support** from **automated decision-making** and explain why the distinction matters.
- Build a simple predictive model on tabular business data.

## Part I — A Framework for Evaluating Any Application

Before the survey, the tool. For any AI application you encounter — in a paper, a product demo, or a news article — ask these five questions:

1. **What is the task type?** (Unit 2 — classification, regression, generation, ranking...)
2. **Where does the training data come from, and who is in it?**
3. **What happens when it is wrong?** Are false positives and false negatives equally costly?
4. **Is it advising a human, or deciding on its own?**
5. **Who is accountable for the outcome?**

```{note}
Question 4 is the one most often glossed over in marketing material. A tool that flags suspicious scans for a radiologist's review is a fundamentally different product — legally, ethically, and practically — from one that issues a diagnosis. Vendors frequently blur this line.
```

## Part II — The Domains

### 2.1 Healthcare

- **Medical imaging** — detecting tumors, fractures, retinal disease. Among the most mature applications.
- **Drug discovery** — protein structure prediction (AlphaFold) has genuinely accelerated research.
- **Clinical documentation** — ambient transcription of doctor-patient visits.
- **Risk prediction** — flagging patients likely to deteriorate or be readmitted.

> **The hard part:** models trained at one hospital often fail at another, because scanner models, patient demographics, and labeling practices differ. This is **distribution shift** (Unit 5), and in medicine it can be lethal.

### 2.2 Education

- Adaptive learning platforms that adjust difficulty to the student
- Automated grading and feedback on written work
- Early-warning systems identifying students at risk
- AI tutors available at any hour

> **The hard part:** early-warning systems trained on historical outcomes learn historical inequities. A model predicting "likely to drop out" may simply be re-encoding which students received less support in the past. Unit 10 makes this concrete.

### 2.3 Law

- **E-discovery** — sorting millions of documents for relevance (a mature, well-accepted use)
- Contract review and clause extraction
- Legal research and case retrieval
- **Risk assessment in sentencing and bail** — deeply contested

> **The hard part:** generative models fabricate citations. Multiple attorneys have been sanctioned for filing briefs containing cases that do not exist. Verification is not optional.

### 2.4 Business and Finance

- Demand forecasting and inventory optimization
- Fraud detection — one of the oldest and most successful applications
- Credit scoring and underwriting — heavily regulated, heavily scrutinized
- Algorithmic trading
- Customer service automation

### 2.5 Creative Industries

- Text, image, audio, and video generation
- Design iteration and concept exploration
- Music composition and production tools
- Game asset generation

> **The hard part:** these questions are unresolved. Who owns output trained on copyrighted work? What happens to entry-level creative jobs? These are live legal and economic disputes, not settled matters — see Unit 12.

## ⚙️ Hands-On: A Business Forecasting Model

Below is a compact but realistic application: predicting monthly sales from marketing spend and seasonality. This is **regression** (Unit 2) applied to tabular business data — by far the most common form of AI in industry, and the least glamorous.

```python
import numpy as np, pandas as pd, matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_absolute_error, r2_score

rng = np.random.default_rng(11)
n = 240

# --- Simulate three years of monthly store data -------------------
df = pd.DataFrame({
    "ad_spend":    rng.uniform(5, 60, n).round(1),      # thousands $
    "month":       rng.integers(1, 13, n),
    "competitors": rng.integers(0, 6, n),
    "foot_traffic": rng.normal(1500, 300, n).round(),
})
# Sales depend on spend (with diminishing returns), season, competition
season = 1 + 0.35 * np.sin((df["month"] - 3) / 12 * 2 * np.pi)
df["sales"] = (12 * np.sqrt(df["ad_spend"]) * season
               - 4 * df["competitors"]
               + 0.02 * df["foot_traffic"]
               + rng.normal(0, 6, n)).round(1)

print(df.head(), "\n")

X = df.drop(columns="sales"); y = df["sales"]
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.25, random_state=11)

for name, model in [("Linear Regression", LinearRegression()),
                    ("Random Forest", RandomForestRegressor(
                        n_estimators=200, random_state=11))]:
    model.fit(X_tr, y_tr)
    pred = model.predict(X_te)
    print(f"{name:20s} MAE {mean_absolute_error(y_te, pred):6.2f}   "
          f"R² {r2_score(y_te, pred):.3f}")

# Which factors drive sales? (Random Forest can tell us)
rf = RandomForestRegressor(n_estimators=200, random_state=11).fit(X_tr, y_tr)
imp = pd.Series(rf.feature_importances_, index=X.columns).sort_values()

plt.figure(figsize=(8, 4))
imp.plot(kind="barh", color="seagreen")
plt.xlabel("Relative importance")
plt.title("What drives sales, according to the model?", weight="bold")
plt.tight_layout(); plt.show()
```

The Random Forest outperforms Linear Regression here because the true relationship involves a square root and a seasonal wave — curves that a straight line cannot follow.

```{warning}
**Feature importance is not causation.** The chart tells you which variables the model *used*, not which ones *cause* sales. If rainy days both reduce foot traffic and increase online orders, the model may show foot traffic as important while being completely wrong about the mechanism. Acting on a correlation as though it were a cause is the most common and most expensive mistake in applied AI.
```

**Try changing it:**

1. Add a `store_id` column of random numbers and refit. Does the model assign it importance? What does spurious importance tell you about trusting these charts?
2. Remove `ad_spend` entirely. How much does performance drop? That gap estimates its real contribution.
3. Frame this as a **decision**: at what point should a manager act on this model without a human review? Use the five questions from Part I.

## 💡 Example: Applying the Framework

Take the claim: *"Our AI reduces hospital readmissions by 20%."*

| Question | What you should ask |
| :--- | :--- |
| **Task type?** | Classification — will this patient return within 30 days? |
| **Training data?** | Which hospital, which years, which patient population? |
| **Cost of errors?** | A missed high-risk patient vs. wasted follow-up resources |
| **Advisory or automated?** | Does it flag for a nurse, or auto-assign discharge plans? |
| **Accountable?** | The hospital, the vendor, or the clinician who accepted the flag? |

Notice that **none of these questions is about accuracy.** The accuracy number is the easiest thing to report and the least informative thing to know.

## 🧭 Reflection

> Across all six domains, the mature and uncontroversial applications share a trait: **the AI narrows a search space and a human makes the decision.** The contested applications are those where the system decides.
> Is that a permanent boundary, or just where we currently are?

**Connecting to HW6 (AI Application Analysis — Choose a Domain, due Oct 21):** pick a domain and a specific claimed application, then run it through the five questions. Depth on one application will earn more than breadth across many.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _AI: A Modern Approach_, 4th Ed., Ch. 27. Pearson.
- Topol, E. (2019). _Deep Medicine: How Artificial Intelligence Can Make Healthcare Human Again._ Basic Books.
- Dendritic Institute (2025). _AI Literacy Series — Module 8: Case Studies._
