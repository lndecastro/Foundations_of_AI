# Unit 16: Responsible AI

> **Sessions 24–25** · Nov 09, 16 · **HW12 assigned Nov 09, due Nov 16**

This unit closes the course, and it is a synthesis rather than a new topic. Data quality, privacy, bias, fairness, ethics, and governance have appeared throughout Parts IV and V — confidentiality in Unit 10, fabricated numbers in Unit 12, confounding in Unit 13, honest axes in Unit 14, licensing and security in Unit 15. Here they become a single framework you can apply deliberately.

Two sessions is not much time for material that fills entire degree programs. The goal is correspondingly focused: **you should leave able to identify what could go wrong with a specific system, in specific terms, and say who would be harmed.** That is more useful than a survey of ethical theories.

## Learning Objectives

After completing this unit, you will be able to:

- Detect **data leakage** and explain why it inflates results silently.
- Demonstrate that anonymization is weaker than it appears.
- Distinguish sources of bias and show that fairness definitions can conflict mathematically.
- Apply an accountability framework and produce a **model card**.
- Locate a system within the current regulatory landscape.

## Part I — Data: Quality, Leakage, Privacy

### 1.1 The Quality Dimensions

| Dimension | Question |
| :--- | :--- |
| Accuracy | Do values reflect reality? |
| Completeness | What is missing, and is it missing *at random*? |
| Consistency | Do sources agree? |
| Timeliness | Is it current enough to act on? |
| Provenance | Where did it come from, and with what consent? |

Completeness deserves emphasis. **Missing data is almost never missing at random.** Patients who miss follow-up appointments differ systematically from those who attend. Employees who skip the engagement survey differ from those who complete it. Dropping incomplete rows does not remove a nuisance; it silently changes the population your model describes.

### 1.2 Leakage

Leakage is when information unavailable at prediction time leaks into training. It produces spectacular results that collapse in deployment, and it is the single most common way for a competent team to ship something worthless.

## ⚙️ Hands-On: Leakage in Action

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

rng = np.random.RandomState(5)
n = 1200

# Real clinical factors that genuinely predict readmission
age        = rng.randint(40, 90, n)
prior_adm  = rng.poisson(1.2, n)
comorbid   = rng.randint(0, 6, n)
risk = (0.02 * (age - 40) + 0.30 * prior_adm + 0.15 * comorbid
        + rng.normal(0, 1.0, n))
readmitted = (risk > np.percentile(risk, 70)).astype(int)

# THE LEAK: a field the hospital system only fills in AFTER a readmission.
# A well-meaning analyst pulls the whole table and includes it.
followup_code = np.where(readmitted == 1,
                         rng.choice([31, 32], n),      # post-readmission codes
                         rng.choice([10, 11, 12], n))

df = pd.DataFrame({"age": age, "prior_admissions": prior_adm,
                   "comorbidities": comorbid,
                   "followup_code": followup_code,
                   "readmitted": readmitted})

def evaluate(features, label):
    X_tr, X_te, y_tr, y_te = train_test_split(
        df[features], df.readmitted, test_size=.3,
        random_state=1, stratify=df.readmitted)
    m = RandomForestClassifier(n_estimators=120, random_state=1).fit(X_tr, y_tr)
    acc = accuracy_score(y_te, m.predict(X_te))
    print(f"{label:<34} test accuracy: {acc:.1%}")
    return m, features

clean = ["age", "prior_admissions", "comorbidities"]
leaky = clean + ["followup_code"]

m_leak, f_leak = evaluate(leaky, "WITH the leaked column")
m_ok,   f_ok   = evaluate(clean, "WITHOUT it (honest model)")

print("\nFeature importances of the leaky model:")
for f, i in sorted(zip(f_leak, m_leak.feature_importances_),
                   key=lambda t: -t[1]):
    print(f"  {f:<20}{i:.3f}")
```

The leaky model looks excellent. It is useless: in deployment `followup_code` does not exist yet for the patient you are trying to predict, because it is created *by* the event you are predicting.

The feature-importance listing is the diagnostic. **When one feature dominates implausibly, suspect leakage before celebrating.** The honest model's lower accuracy is the real number, and it is the one that would survive deployment.

**Try changing it:**

1. Make the leak subtler — have `followup_code` agree with the outcome only 80% of the time. Accuracy drops but stays inflated. Is it still detectable from importances?
2. Add a `days_since_discharge` column that is `NaN` for non-readmitted patients. Does missingness itself leak?
3. Write down the four questions you would ask about **any** feature to decide whether it could leak. This is your checklist.

## ⚙️ Hands-On: Anonymization Is Weaker Than It Looks

```python
import pandas as pd
import numpy as np

rng = np.random.RandomState(7)
n = 5000
zips = [33901, 33907, 33912, 33913, 33916, 33919, 33928, 33931, 33966]

records = pd.DataFrame({
    "zip":        rng.choice(zips, n),
    "birth_year": rng.randint(1940, 2006, n),
    "sex":        rng.choice(["F", "M"], n),
    "diagnosis":  rng.choice(["A", "B", "C", "D"], n),
})
# Names and IDs removed. This is what "de-identified" usually means.

quasi = ["zip", "birth_year", "sex"]
sizes = records.groupby(quasi).size()
unique = (sizes == 1).sum()

print(f"Records: {n}")
print(f"Uniquely identified by ZIP + birth year + sex: "
      f"{unique} ({unique / n:.1%})")
print(f"In groups of 2 or fewer: "
      f"{sizes[sizes <= 2].sum()} ({sizes[sizes <= 2].sum() / n:.1%})")

# k-anonymity: generalize birth year into decades
records["birth_decade"] = (records.birth_year // 10) * 10
sizes_k = records.groupby(["zip", "birth_decade", "sex"]).size()
print(f"\nAfter generalizing birth year to decade:")
print(f"  smallest group size (k): {sizes_k.min()}")
print(f"  uniquely identified:     {(sizes_k == 1).sum()}")
```

Names removed, IDs removed — and dozens of records are still uniquely identified by three fields nobody considers identifying, with hundreds more in groups of two. Anyone holding a voter roll can match them. Note that this is a *generous* test: only nine ZIP codes and 5,000 records. Widen the ZIP range or narrow the population and the unique fraction climbs sharply, which is why real re-identification studies on state-level data reach far higher numbers.

Generalizing birth year to a decade raises **k-anonymity** sharply. It also destroys information, and that trade-off is the entire field: privacy protection is paid for in analytical utility, and someone has to decide the exchange rate.

```{warning}
"We removed personally identifiable information" is a description of an action, not a guarantee of an outcome. The correct question is always: **what is the smallest group in this dataset, on any combination of fields an adversary might hold?**
```

## Part II — Bias and Fairness

### 2.1 Where Bias Enters

| Source | Mechanism |
| :--- | :--- |
| **Historical** | The data faithfully records a biased world |
| **Representation** | Some groups are undersampled |
| **Measurement** | The proxy differs in meaning across groups |
| **Aggregation** | One model imposed where groups differ |
| **Deployment** | Used on a population it was not trained for |

Historical bias is the hardest because the data is *accurate*. A hiring model trained on ten years of decisions learns those decisions. If they were biased, the model reproduces the bias — faithfully, at scale, and with the appearance of objectivity that a number carries.

Measurement bias is the subtlest. Healthcare cost has been used as a proxy for health need; because less is historically spent on some populations for the same condition, the proxy encodes unequal access as lower need. Nothing in the pipeline is broken. The proxy is simply not measuring what it was assumed to measure.

### 2.2 Fairness Definitions Conflict

This is the mathematically important result of the unit, and it is not a matter of opinion.

- **Demographic parity** — equal positive rates across groups.
- **Equal opportunity** — equal true-positive rates across groups.
- **Predictive parity** — equal precision across groups.
- **Individual fairness** — similar individuals treated similarly.

**When base rates genuinely differ between groups, these cannot all hold simultaneously.** Not "are hard to achieve" — cannot. This was proved in the context of the COMPAS recidivism debate, where a system was defended as fair by one definition and attacked as unfair by another, and both sides were arithmetically right.

```{warning}
The consequence is that **there is no technically correct answer to "is this system fair?"** Choosing a fairness criterion is choosing which harm to prioritize, and that is a normative decision requiring the people affected to have a say.

An engineering team that picks a fairness metric quietly, because it needed one to proceed, has made a policy decision without acknowledging it. Naming the choice is the professional obligation.
```

## Part III — Accountability and Governance

### 3.1 The Accountability Gap

When an AI system causes harm, responsibility diffuses: the data was collected by one party, the model built by another, the product designed by a third, deployed by a fourth, and used by a fifth. Everyone acted reasonably within their scope, and nobody owns the outcome.

Closing the gap requires naming, in advance, who is answerable for what. The mechanism is documentation.

### 3.2 Model Cards

```python
model_card = {
    "name": "Sprint Completion Predictor v0.3",
    "owner": "your name and contact",
    "date": "2026-11-09",

    "intended_use": "Flag sprints at risk of under-delivery for team discussion.",
    "out_of_scope": [
        "Individual performance evaluation",
        "Any input to compensation or promotion decisions",
        "Teams outside the two it was trained on",
    ],

    "training_data": {
        "source": "Internal sprint records, Jan 2025 - Sep 2026",
        "size": "1,340 sprint-engineer rows",
        "known_gaps": "Two teams only; no contractor data; "
                      "task difficulty not recorded",
        "consent": "Team leads informed; individual engineers not consulted",
    },

    "performance": {
        "overall": "accuracy 0.78, precision 0.71, recall 0.66",
        "subgroups": "Not evaluated by team size — a known gap",
        "baseline": "Always-predict-on-time: 0.64 accuracy",
    },

    "known_limitations": [
        "Trained on a period including one reorganization",
        "Story points are not comparable across teams",
        "Does not distinguish under-delivery from over-commitment",
    ],

    "fairness": {
        "criterion_chosen": "Equal opportunity across teams",
        "why": "False negatives (missed risk) are the costly error here",
        "not_chosen": "Demographic parity — teams have different real risk",
    },

    "accountable_party": "who is answerable if this causes harm",
}

def render(card, indent=0):
    for k, v in card.items():
        pad = "  " * indent
        if isinstance(v, dict):
            print(f"{pad}{k.upper().replace('_', ' ')}:"); render(v, indent + 1)
        elif isinstance(v, list):
            print(f"{pad}{k.replace('_', ' ')}:")
            for item in v:
                print(f"{pad}  - {item}")
        else:
            print(f"{pad}{k.replace('_', ' ')}: {v}")

render(model_card)
```

**Try changing it:**

1. Fill this in for a model you built in Unit 4 or your capstone. Which field was hardest to complete honestly?
2. The `baseline` field is the one most often omitted. Why does 0.78 accuracy mean something different once you know the trivial baseline is 0.64?
3. Add a `monitoring` section: what would you measure after deployment, and what result would trigger retraining?

### 3.3 Prompts for Responsible AI Work

These three do most of the work on HW12. Full versions with verification steps are in Appendix D.10.

**Leakage audit** — run this on your capstone's feature list before you model anything:

> Below is a list of the features in my dataset and a description of the outcome I am predicting. For each feature, tell me: could this value be unavailable, incomplete, or different at the moment a real prediction would be made? Consider fields that are populated after the outcome, fields derived from the outcome, and fields whose meaning changes over the record's lifecycle. Flag anything suspicious even if you are unsure — I would rather check five clean features than miss one leak.
> OUTCOME: «what you are predicting, and when the prediction happens»
> FEATURES: «list with a one-line description of each»

**Bias by mechanism** — the phrasing matters, because a vague prompt returns vague risks:

> Identify sources of bias by mechanism, using these categories: historical, representation, measurement, aggregation, deployment. For each one you identify, state the specific mechanism — not that bias "could exist," but what in this particular pipeline would produce it, and which group would be affected how. If a category does not apply here, say so rather than inventing an instance.
> DATASET: «source, collection period, population, label definition»
> DECISION: «what the output is used for, and by whom»

**Fairness trade-off:**

> For the system described below, walk through what each of these fairness criteria would require: demographic parity, equal opportunity, predictive parity, individual fairness. Then tell me which pairs cannot hold simultaneously given the base rates I describe, and what each choice sacrifices. Do not recommend one — state what a person choosing each would be prioritizing.
> SYSTEM: «what it decides, for whom» · BASE RATES: «outcome rates across groups» · ERROR COSTS: «what each error costs, and to whom»

```{note}
Note the shape shared by all three: they forbid the reassuring answer. "Could be biased," "seems reasonable," and "no major concerns" are the statistically typical continuations, and they are worth nothing on this assignment. Constraining the output away from them is the whole technique.
```

### 3.4 The Regulatory Landscape

- **EU AI Act** — risk-tiered. Unacceptable-risk uses prohibited; high-risk uses (employment, credit, education, law enforcement) face conformity, documentation, and human-oversight requirements. Extraterritorial in effect.
- **United States** — sectoral rather than comprehensive. FTC authority over unfair practices, sector regulators in finance and health, plus state laws — Colorado's AI Act and Illinois's biometric statute among the most consequential.
- **NIST AI Risk Management Framework** — voluntary in the US, and rapidly becoming the de facto documentation standard.
- **ISO/IEC 42001** — an auditable AI management-system standard.

```{note}
The practical takeaway for a software engineer is narrower than the landscape suggests: **know whether what you are building falls in a high-risk category.** Employment, credit, education, healthcare, and law enforcement applications carry obligations that a recommendation engine for music does not. The classification determines the paperwork, and the paperwork determines the timeline.
```

## 💡 Example: One System, Four Views

A university deploys a model flagging students at risk of failing, so advisors can intervene.

- **Data view.** The label is "failed the course." Historical grades encode past instructor variation. Students who withdrew are missing — not at random.
- **Fairness view.** Equal opportunity across demographic groups? Or equal precision, so that advisors' limited time is not wasted disproportionately on one group? These conflict, and choosing is a policy decision.
- **Accountability view.** If a flagged student is treated differently and performs worse as a result, who is answerable — the vendor, the institution, or the advisor who acted on the flag?
- **Governance view.** Educational applications are high-risk under the EU AI Act. Under FERPA, the data has consent constraints that most vendors' terms do not satisfy.

Every one of these is a real objection, and none is resolved by improving the model's accuracy.

## 🧭 Reflection

> Across this course you built classifiers, generated documents, analyzed data, and evaluated coding tools. In nearly every unit, the failure mode was the same shape: **the system produced something that looked right.**
>
> What is the one habit you will actually keep — not the one you should keep — that would catch that?

**Connecting to HW12 (Responsible AI Assessment):** assigned Nov 09, due Nov 16. Take one AI system — ideally your own capstone — and produce a completed model card plus a one-page risk assessment covering: one leakage risk, one privacy risk, one bias source with its mechanism, the fairness criterion you chose and what you gave up, and who is accountable. Vague risks ("could be biased") receive no credit; name the mechanism.

**Capstone presentations begin Nov 18.**

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 27. Pearson.
- Mitchell, M. (2020). _Artificial Intelligence: A Guide for Thinking Humans_, Ch. 7, Part V. Penguin Books.
- Barocas, S., Hardt, M., & Narayanan, A. (2023). _Fairness and Machine Learning_. MIT Press. <https://fairmlbook.org>
- Mitchell, M., et al. (2019). _Model Cards for Model Reporting._ ACM FAccT.
- Sweeney, L. (2002). _k-Anonymity: A Model for Protecting Privacy._ IJUFKS 10(5).
- NIST (2023). _AI Risk Management Framework 1.0._
