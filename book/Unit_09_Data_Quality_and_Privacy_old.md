# Unit 9: Data, Quality and Privacy

> **Sessions 22–23** · Nov 02, Nov 04 · **HW9 assigned Nov 04, due Nov 09**

Unit 3 claimed that data collection and preprocessing consume roughly 60% of the effort in an AI project. This unit shows you why — and demonstrates two failures that destroy more real-world projects than any modeling mistake: **data leakage** and **failed anonymization**.

## Learning Objectives

After completing this unit, you will be able to:

- Assess **data quality** along the standard dimensions and identify common defects.
- Recognize and diagnose **data leakage**, the most expensive silent failure in machine learning.
- Explain why **removing names does not anonymize data**, and demonstrate re-identification.
- Describe the major **data ownership and consent** questions facing AI systems.

## Part I — Data Quality

### 1.1 The Quality Dimensions

| Dimension | Question | Typical failure |
| :--- | :--- | :--- |
| **Completeness** | Are values missing? | Sensor gaps, skipped survey fields |
| **Accuracy** | Are values correct? | Transcription errors, broken instruments |
| **Consistency** | Do values agree across sources? | Same patient, two birthdates |
| **Timeliness** | Is the data current? | Model trained on pre-pandemic behavior |
| **Representativeness** | Does it reflect the deployment population? | Trained in one city, deployed nationally |
| **Provenance** | Do we know where it came from? | Scraped data of unknown origin |

> **Representativeness is the one that produces headlines.** A model is only valid for populations resembling its training data. Unit 10 traces what happens when that assumption fails along demographic lines.

### 1.2 Missing Data Is Not Random

The reflexive response to missing values is to fill them with the average. This is frequently wrong, because **the reason data is missing usually carries information.**

- Patients who skip a follow-up may be those who recovered — or those who died.
- Customers who leave income blank may differ systematically from those who don't.
- A sensor that fails in extreme heat leaves gaps precisely during the conditions you care about.

Filling those gaps with an average erases the signal and replaces it with a fabrication.

## ⚙️ Hands-On 1: Data Leakage — The Silent Project Killer

**Data leakage** occurs when information that would not be available at prediction time sneaks into training. The symptom is suspiciously excellent performance. The consequence is a model that fails completely in production.

```python
import numpy as np, pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score

rng = np.random.default_rng(5)
n = 800

# Hospital data: predict whether a patient will be readmitted
df = pd.DataFrame({
    "age":            rng.integers(20, 90, n),
    "prior_visits":   rng.poisson(2, n),
    "length_of_stay": rng.integers(1, 15, n),
    "bmi":            rng.normal(27, 5, n).round(1),
})
# The true outcome depends on real clinical factors
risk = (0.02 * df["age"] + 0.35 * df["prior_visits"]
        + 0.08 * df["length_of_stay"] - 2.8 + rng.normal(0, 0.7, n))
df["readmitted"] = (risk > 0).astype(int)

# --- THE LEAK -----------------------------------------------------
# A well-meaning analyst includes this column from the hospital system.
# It is only ever filled in AFTER a readmission happens.
df["followup_appt_scheduled"] = np.where(
    df["readmitted"] == 1,
    rng.random(n) < 0.92,        # nearly always scheduled if readmitted
    rng.random(n) < 0.08         # rarely otherwise
).astype(int)
# ------------------------------------------------------------------

def evaluate(features, label):
    X = df[features]; y = df["readmitted"]
    X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.3, random_state=5)
    m = LogisticRegression(max_iter=1000).fit(X_tr, y_tr)
    acc = accuracy_score(y_te, m.predict(X_te))
    print(f"{label:35s} {acc:.1%}")

clean  = ["age", "prior_visits", "length_of_stay", "bmi"]
leaked = clean + ["followup_appt_scheduled"]

evaluate(clean,  "Honest model (clean features)")
evaluate(leaked, "Model WITH leaked feature")
```

The leaked model scores far higher. It looks like a triumph. **It is worthless**, because at the moment you need a prediction — when the patient is being discharged — the follow-up appointment does not yet exist. The model has learned to read the answer off the back of the page.

```{warning}
**How to catch leakage.** Ask one question of every feature: *would this value be available, in this form, at the exact moment I need the prediction?* If the answer is no or unclear, remove it. Suspiciously high accuracy should trigger suspicion, not celebration. Leakage has killed deployed systems at major hospitals, banks, and tech companies — it is not a beginner's mistake.
```

**Try changing it:**

1. Weaken the leak (change 0.92 to 0.65). Does it still inflate accuracy? Subtle leaks are the dangerous ones because they don't look obviously wrong.
2. Add a leaked feature of your own invention to this dataset.
3. For your **capstone**, list every feature and answer the availability question for each. Do this before you build anything.

## Part II — Privacy

### 2.1 Anonymization Is Harder Than It Looks

Removing names and ID numbers feels like anonymization. It is not. **Combinations of ordinary attributes are frequently unique**, and unique means identifiable.

Landmark research established that a large share of the US population can be uniquely identified from just **ZIP code, birth date, and sex** — none of which are considered identifying on their own.

## ⚙️ Hands-On 2: Re-identification

```python
import numpy as np, pandas as pd

rng = np.random.default_rng(3)
n = 5000

# Real ZIP codes from Lee County, Florida
ZIPS = [33901, 33903, 33904, 33905, 33907, 33908, 33909, 33912, 33913,
        33914, 33916, 33917, 33919, 33928, 33931, 33966, 33967, 33971,
        33972, 33973, 33974, 33976, 33990, 33991, 33993, 34134, 34135]

# A "de-identified" medical release: names and IDs removed
df = pd.DataFrame({
    "zip":        rng.choice(ZIPS, n),
    "birth_year": rng.integers(1935, 2007, n),
    "sex":        rng.choice(["M", "F"], n),
    "diagnosis":  rng.choice(["asthma", "diabetes", "hypertension",
                              "migraine", "depression"], n),
})

# How many people are UNIQUE on the three "non-identifying" fields?
quasi = ["zip", "birth_year", "sex"]
group_sizes = df.groupby(quasi)[quasi[0]].transform("size")

unique       = (group_sizes == 1).sum()
small_groups = (group_sizes <= 3).sum()

print(f"Records in the release:                 {n}")
print(f"Uniquely identifiable by ZIP+year+sex:  {unique}  ({unique/n:.1%})")
print(f"In a group of 3 or fewer:               {small_groups}  ({small_groups/n:.1%})")
print("\nFor anyone in the unique group, knowing those three")
print("public facts reveals their medical diagnosis.\n")

# k-anonymity: generalize birth year into 10-year bands
df["birth_decade"] = (df["birth_year"] // 10) * 10
quasi2 = ["zip", "birth_decade", "sex"]
sizes2 = df.groupby(quasi2)[quasi2[0]].transform("size")
print(f"After generalizing birth year to decade:")
print(f"  Uniquely identifiable: {(sizes2 == 1).sum()}  "
      f"(k-anonymity k = {sizes2.min()})")
```

Run it. In this release, roughly **28% of records are uniquely identifiable** and **87% sit in groups of three or fewer** — on three fields that no one would call identifying, in a county-sized population. Anyone who knows their neighbor's ZIP code, approximate age, and sex has a good chance of learning their diagnosis. After generalizing birth year into decades, uniqueness drops to zero.

> **This is the privacy trade-off in its purest form.** Generalization protects people and destroys information. There is no setting that gives you both — only a choice about where to sit, and that choice is ethical, not technical.

**Try changing it:**

1. Add `income_bracket` as a fourth quasi-identifier. Uniqueness rises sharply — **each additional attribute compounds the risk.**
2. Reduce `n` to 500. Small datasets are far more re-identifiable. What does that mean for research on rare diseases?
3. Try generalizing ZIP to the first three digits instead. Which generalization costs more analytical value?

### 2.2 Ownership, Consent, and Provenance

Three questions with no settled answers:

- **Who owns training data?** Text and images scraped from the open web were not licensed for this purpose. Litigation is ongoing worldwide.
- **What counts as consent?** Data collected for one purpose is routinely repurposed for model training years later. Did anyone agree to that?
- **Can data be withdrawn?** Deleting a record from a database does not remove its influence from a trained model. "Machine unlearning" is an active and unsolved research area.

### 2.3 Regulatory Landscape

| Framework | Scope | Key provision |
| :--- | :--- | :--- |
| **GDPR** (EU) | Personal data of EU residents | Consent, erasure, explanation of automated decisions |
| **HIPAA** (US) | Health information | Restricts use and disclosure of PHI |
| **FERPA** (US) | Student education records | Governs institutional data sharing |
| **CCPA/CPRA** (California) | Consumer data | Disclosure and opt-out rights |

Unit 12 examines how these interact with AI-specific regulation.

## 🧭 Reflection

> The leakage model was more accurate and completely useless. What does that say about accuracy as a criterion for trust?
>
> If your medical records were released with names removed, and you could be re-identified from your ZIP code and birthday — were you ever meaningfully anonymous?

**Connecting to HW9 (Data Privacy Case Study, due Nov 09):** find a real re-identification or data breach incident and analyze it using this unit's framework. What quasi-identifiers were involved? What generalization would have prevented it, and at what analytical cost?

## 📘 Further Reading

- Sweeney, L. (2002). k-anonymity: A Model for Protecting Privacy. _International Journal of Uncertainty, Fuzziness and Knowledge-Based Systems_, 10(5).
- Narayanan, A., & Shmatikov, V. (2008). Robust De-anonymization of Large Sparse Datasets. _IEEE Symposium on Security and Privacy._
- Dwork, C., & Roth, A. (2014). _The Algorithmic Foundations of Differential Privacy._
- Russell, S., & Norvig, P. (2022). _AI: A Modern Approach_, 4th Ed., §27.3. Pearson.
