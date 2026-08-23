# Unit 10: Bias and Fairness

> **Sessions 24–25** · Nov 09, Nov 16 · **HW10 assigned Nov 16, due Nov 23**
> *(Note: Nov 11 is Veterans Day — no class.)*

This unit contains the most consequential experiment in the course. You will build a hiring model that is **statistically excellent and discriminatory at the same time** — without ever giving it access to gender. Then you will try to fix it, and discover that the standard fairness criteria are mathematically incompatible with each other.

## Learning Objectives

After completing this unit, you will be able to:

- Identify **where bias enters** an AI pipeline, at each of the six lifecycle stages.
- Explain why **removing a protected attribute does not remove bias**.
- Compute and interpret common **fairness metrics**.
- Explain why fairness criteria **cannot generally be satisfied simultaneously**.

## Part I — Where Bias Enters

Bias is not a bug injected at one point. It can enter at every stage of Unit 3's pipeline.

| Stage | How bias enters | Example |
| :--- | :--- | :--- |
| **1. Problem framing** | Choosing the wrong target variable | Predicting "arrests" as a proxy for "crime" |
| **2. Data collection** | Unrepresentative sampling | Facial datasets dominated by lighter skin tones |
| **3. Preprocessing** | Dropping records with missing data | Excludes populations with less complete records |
| **4. Training** | Optimizing average accuracy | Model sacrifices minority-group performance for overall score |
| **5. Evaluation** | Reporting only aggregate metrics | 95% overall accuracy conceals 60% for one subgroup |
| **6. Deployment** | Feedback loops | Predictions shape reality, which becomes future training data |

```{important}
**Stage 1 is the most overlooked and often the most damaging.** No amount of technical fairness work can repair a badly chosen target variable. If you predict "who was arrested" and call it "who commits crime," you have encoded historical policing patterns into the definition of the problem, and every downstream fix is cosmetic.
```

### 1.1 Proxy Variables

The most common misconception in this area: *"We'll just not give the model gender or race, and then it can't discriminate."*

This does not work. Protected attributes are **correlated with other variables**, and models find those correlations automatically. ZIP code proxies for race. Gaps in employment history proxy for parental status. The name of a school, membership in certain organizations, even writing style — all can carry the signal.

> **Removing a protected attribute removes your ability to measure discrimination. It does not remove the discrimination.**

## ⚙️ Hands-On 1: A Discriminatory Model That Never Sees Gender

```python
import numpy as np, pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

rng = np.random.default_rng(21)
n = 4000

gender = rng.choice(["A", "B"], n)          # two groups
is_B = (gender == "B")

# Genuine, gender-neutral qualification
skill = rng.normal(0, 1, n)

# But the WORLD is not neutral: group B has historically had
# less access to certain credentials and more career gaps.
years_exp   = np.clip(rng.normal(8, 3, n) - 2.5 * is_B, 0, None).round(1)
career_gaps = rng.poisson(0.4 + 1.1 * is_B, n)
elite_school = (rng.random(n) < np.where(is_B, 0.12, 0.30)).astype(int)

# HISTORICAL hiring decisions encode past discrimination
hired_historically = (
    0.9 * skill + 0.10 * years_exp - 0.35 * career_gaps
    + 0.6 * elite_school - 0.9 * is_B          # <-- past bias
    + rng.normal(0, 0.6, n)
) > 1.0

df = pd.DataFrame({
    "years_exp": years_exp, "career_gaps": career_gaps,
    "elite_school": elite_school, "skill_score": skill.round(2),
    "hired": hired_historically.astype(int), "gender": gender,
})

# --- Train WITHOUT gender. The model literally cannot see it. -----
FEATURES = ["years_exp", "career_gaps", "elite_school", "skill_score"]
X_tr, X_te, y_tr, y_te, g_tr, g_te = train_test_split(
    df[FEATURES], df["hired"], df["gender"], test_size=0.3, random_state=21)

model = RandomForestClassifier(n_estimators=200, random_state=21).fit(X_tr, y_tr)
pred = model.predict(X_te)

print(f"Overall accuracy: {accuracy_score(y_te, pred):.1%}")
print("Gender was NOT a feature. Now look at the outcomes:\n")

for g in ["A", "B"]:
    mask = (g_te == g).values
    rate = pred[mask].mean()
    print(f"  Group {g}: recommended for hire {rate:.1%} of the time")

rate_A = pred[(g_te == 'A').values].mean()
rate_B = pred[(g_te == 'B').values].mean()
print(f"\n  Selection-rate ratio (B/A): {rate_B/rate_A:.2f}")
print("  US EEOC 'four-fifths rule' flags anything below 0.80")
```

Run it. The model achieves strong accuracy and produces a large disparity between groups — **while being structurally incapable of seeing gender.** It learned the proxies.

## ⚙️ Hands-On 2: Fairness Metrics and Their Incompatibility

There is no single definition of fairness. There are several, each defensible, and **they conflict mathematically.**

```python
import numpy as np, pandas as pd

def fairness_report(y_true, y_pred, groups):
    rows = []
    for g in np.unique(groups):
        m = (groups == g)
        yt, yp = np.asarray(y_true)[m], np.asarray(y_pred)[m]
        tp = ((yp == 1) & (yt == 1)).sum(); fp = ((yp == 1) & (yt == 0)).sum()
        fn = ((yp == 0) & (yt == 1)).sum(); tn = ((yp == 0) & (yt == 0)).sum()
        rows.append({
            "group": g,
            "selection_rate": yp.mean(),                       # demographic parity
            "true_positive_rate": tp / max(tp + fn, 1),        # equal opportunity
            "false_positive_rate": fp / max(fp + tn, 1),       # equalized odds
            "precision": tp / max(tp + fp, 1),                 # predictive parity
            "accuracy": (tp + tn) / len(yt),
        })
    return pd.DataFrame(rows).set_index("group").round(3)

report = fairness_report(y_te.values, pred, g_te.values)
print(report, "\n")

print("Which definition of 'fair' do you want?")
print(f"  Demographic parity  -> equal selection rates:  "
      f"{report['selection_rate'].min()/report['selection_rate'].max():.2f}")
print(f"  Equal opportunity   -> equal TPR:              "
      f"{report['true_positive_rate'].min()/report['true_positive_rate'].max():.2f}")
print(f"  Predictive parity   -> equal precision:        "
      f"{report['precision'].min()/report['precision'].max():.2f}")
print("\n(1.00 = perfectly equal on that criterion)")
```

| Definition | Requirement | Objection |
| :--- | :--- | :--- |
| **Demographic parity** | Equal selection rates across groups | Ignores genuine differences in the outcome |
| **Equal opportunity** | Equal true positive rates | Permits unequal selection rates |
| **Predictive parity** | Equal precision | Can permit unequal error burdens |
| **Individual fairness** | Similar individuals treated similarly | Who defines "similar"? |

```{warning}
**This is a proven impossibility, not an engineering gap.** Kleinberg, Mullainathan, and Raghavan (2016) and Chouldechova (2017) showed that when base rates genuinely differ between groups, **calibration, equal false-positive rates, and equal false-negative rates cannot all hold simultaneously** except in degenerate cases.

The practical consequence: *"make the model fair"* is not a well-specified instruction. Someone must choose which fairness criterion governs, and that choice is a value judgment about who bears the cost of error. It cannot be delegated to a data scientist, and it certainly cannot be delegated to the model.
```

## ⚙️ Hands-On 3: Attempting a Fix

```python
# Attempt 1: remove the most obviously proxy-laden feature
FEATURES_2 = ["skill_score", "years_exp"]     # drop gaps + elite school
m2 = RandomForestClassifier(n_estimators=200, random_state=21).fit(
    X_tr[FEATURES_2], y_tr)
pred2 = m2.predict(X_te[FEATURES_2])

r2 = fairness_report(y_te.values, pred2, g_te.values)
print("After dropping career_gaps and elite_school:")
print(r2[["selection_rate", "true_positive_rate", "accuracy"]], "\n")

# Attempt 2: train on skill alone -- the only truly neutral signal
m3 = RandomForestClassifier(n_estimators=200, random_state=21).fit(
    X_tr[["skill_score"]], y_tr)
pred3 = m3.predict(X_te[["skill_score"]])
r3 = fairness_report(y_te.values, pred3, g_te.values)
print("Using skill_score alone:")
print(r3[["selection_rate", "true_positive_rate", "accuracy"]])
```

Dropping proxies narrows the gap and **costs accuracy**. And notice the deeper problem: even the "neutral" model is trained on `hired` — a label produced by a biased historical process. **A model trained on discriminatory decisions learns to reproduce them**, however carefully you select features.

> The only genuine fix is upstream: a better target variable, better data, or a decision not to automate this at all. That conclusion is uncomfortable, and it is correct.

**Try changing it:**

1. Change `- 0.9 * is_B` in the historical labels to `0`. Now the past was fair. Does the model still discriminate? What does that isolate?
2. Adjust the decision threshold separately per group to equalize selection rates. This is **affirmative action implemented in code** — legally fraught in the US. Should it be?
3. Compute the cost: how much accuracy must you surrender for each point of fairness gained?

## 💡 Example: Feedback Loops

A predictive policing system directs patrols to Neighborhood X. More patrols produce more recorded incidents in X. Those records become next month's training data. The model becomes *more* confident about X.

Nothing about crime changed. **The model's prediction created the evidence for itself.** This is why Stage 6 monitoring must include the question: *is this system changing the world it is measuring?*

## 🧭 Reflection

> You built a discriminatory model without giving it the protected attribute. What does that imply about "colorblind" or "gender-blind" system design?
>
> If fairness definitions are mathematically incompatible, who in an organization should make the choice among them — and what would make that choice legitimate?

**Connecting to HW10 (Bias in AI — Real-World Example, due Nov 23):** analyze a documented case using this unit's pipeline table. Identify which stage introduced the bias, and which fairness criterion was violated. Naming the stage precisely is what earns credit.

## 📘 Further Reading

- Barocas, S., Hardt, M., & Narayanan, A. (2023). _Fairness and Machine Learning: Limitations and Opportunities._ MIT Press. (Free online)
- Kleinberg, J., Mullainathan, S., & Raghavan, M. (2016). Inherent Trade-Offs in the Fair Determination of Risk Scores. _arXiv:1609.05807_.
- Buolamwini, J., & Gebru, T. (2018). Gender Shades. _Proceedings of Machine Learning Research_, 81.
- O'Neil, C. (2016). _Weapons of Math Destruction._ Crown.
