# Unit 11: Ethics and Responsible AI

> **Session 26** · Nov 18 · **HW11 assigned Nov 18, due Nov 23**

Unit 10 ended with an uncomfortable result: fairness cannot be fully specified mathematically, so someone must make a value judgment. This unit is about how to make such judgments **well** — deliberately, transparently, and with the reasoning on record.

## Learning Objectives

After completing this unit, you will be able to:

- Apply the major **ethical frameworks** to AI decisions and see where they diverge.
- Use the widely adopted **principles of responsible AI** as an evaluation checklist.
- Distinguish **accountability** from blame and identify who holds it.
- Produce a **model card** documenting a system's intended use and limitations.

## Part I — Frameworks

Ethical disagreements about AI are often not disagreements about facts. They are disagreements about which framework applies.

| Framework | Core question | Applied to an AI triage system |
| :--- | :--- | :--- |
| **Consequentialist** | Which outcome produces the most good? | Deploy if it saves more lives on net |
| **Deontological** | What duties and rights are at stake? | Patients have a right to a human decision-maker |
| **Virtue ethics** | What would a good practitioner do? | Would a conscientious clinician rely on this? |
| **Justice / fairness** | How are benefits and burdens distributed? | Who bears the cost when it errs? |
| **Care ethics** | What do relationships and vulnerability demand? | What does the frightened patient need? |

> A system can be **defensible on consequentialist grounds and indefensible on deontological ones**. Recognizing which framework someone is arguing from is half of resolving an ethical dispute.

## Part II — Principles of Responsible AI

Across the OECD, EU, NIST, UNESCO, and most corporate frameworks, the same cluster recurs:

| Principle | What it requires |
| :--- | :--- |
| **Transparency** | People know an AI is involved and roughly how it works |
| **Explainability** | Decisions can be justified in human terms |
| **Fairness** | Benefits and harms are not unjustly distributed |
| **Privacy** | Data is collected and used with consent and protection |
| **Safety & robustness** | It performs reliably, including under stress |
| **Human oversight** | A person can review, override, and intervene |
| **Accountability** | A specific party answers for outcomes |

```{note}
These principles are easy to endorse and hard to operationalize — a gap sometimes called **"ethics washing."** The test of whether an organization takes them seriously is simple: *has a principle ever caused it to not ship something, or to ship it later?* If not, the principles are marketing.
```

### 2.1 Where Principles Collide

Real cases involve genuine trade-offs, not villains:

- **Accuracy vs. explainability** — the most accurate model is often the least interpretable.
- **Privacy vs. fairness** — you cannot audit for bias across groups without collecting group data.
- **Personalization vs. autonomy** — a system that knows you well can serve you or steer you.
- **Safety vs. access** — restricting a capability protects against misuse and denies legitimate use.

> The second point deserves emphasis. **Auditing for discrimination requires collecting exactly the protected attributes that privacy law discourages collecting.** Organizations sometimes cite privacy as a reason they cannot check for bias. Whether that is principle or convenience is a question worth asking.

### 2.2 Accountability

The hardest question in the field: **when an AI system causes harm, who is responsible?**

| Party | Argument for | Argument against |
| :--- | :--- | :--- |
| **Developer** | Built it, knew its limits | Cannot anticipate every deployment |
| **Deploying organization** | Chose to use it, profits from it | May not understand it technically |
| **Operator** | Made the final call | Often lacks power to override in practice |
| **Regulator** | Set the permissible bounds | Reacts slowly to new technology |

```{important}
**"The algorithm decided" is not an answer.** It is an attempt to locate responsibility in a system that cannot bear it. Every deployed AI system reflects a chain of human choices: what to build, what data to use, what threshold to set, what to do when it is uncertain. Accountability follows those choices.
```

## ⚙️ Hands-On: Generating a Model Card

**Model cards** are the industry standard for documenting a system's intended use, performance, and limitations. Producing one forces you to state things that are easier left vague. Run this on any model you have built in this course — and on your capstone.

```python
from textwrap import dedent

def model_card(**kw):
    return dedent(f"""
    ══════════════════════════════════════════════════════════════
     MODEL CARD — {kw['name']}
    ══════════════════════════════════════════════════════════════

     INTENDED USE
       Purpose......... {kw['purpose']}
       Users........... {kw['users']}
       Decision role... {kw['decision_role']}

     OUT OF SCOPE USES
       {kw['out_of_scope']}

     TRAINING DATA
       Source.......... {kw['data_source']}
       Time period..... {kw['data_period']}
       Population...... {kw['data_population']}
       Known gaps...... {kw['data_gaps']}

     PERFORMANCE
       Overall......... {kw['performance']}
       By subgroup..... {kw['subgroup_performance']}
       Metric choice... {kw['metric_rationale']}

     LIMITATIONS & FAILURE MODES
       {kw['limitations']}

     ETHICAL CONSIDERATIONS
       Fairness def.... {kw['fairness_definition']}
       Human oversight. {kw['oversight']}
       Accountable party {kw['accountable']}

     MAINTENANCE
       Retraining...... {kw['retraining']}
       Monitoring...... {kw['monitoring']}
    ══════════════════════════════════════════════════════════════
    """)

# --- EDIT EVERY FIELD for a model you built in this course --------
print(model_card(
    name="Hospital Readmission Risk Classifier (Unit 9)",
    purpose="Flag patients at elevated 30-day readmission risk for care-team review",
    users="Discharge planning nurses",
    decision_role="ADVISORY ONLY — does not determine discharge",
    out_of_scope="Insurance pricing; staffing decisions; any use without clinician review",
    data_source="Single academic medical center, EHR extract",
    data_period="2019-2023 (includes pandemic disruption)",
    data_population="Adults 18+, predominantly urban, insured",
    data_gaps="Uninsured patients underrepresented; pediatric excluded",
    performance="80.4% accuracy on held-out test set",
    subgroup_performance="NOT YET MEASURED — this is a blocking gap",
    metric_rationale="Recall prioritized: missing a high-risk patient is costlier than a false alarm",
    limitations="Trained at one site; may not transfer. Degrades under distribution shift. "
                "Confidently wrong on out-of-distribution cases.",
    fairness_definition="Equal opportunity (equal true-positive rates across groups)",
    oversight="Nurse reviews every flag; can dismiss without justification",
    accountable="Chief Medical Information Officer",
    retraining="Quarterly, with drift monitoring",
    monitoring="Monthly subgroup performance audit; alert if any group TPR drops >5%",
))
```

**Try changing it:**

1. Fill this out for your **capstone project**. Whichever field you find hardest to complete honestly is pointing at the weakest part of your proposal.
2. Notice `subgroup_performance` is marked as a blocking gap. Should a model ship without it? Under what circumstances?
3. Try filling out `accountable` with a *role* rather than a person. Does that weaken the accountability? Compare with `out_of_scope` — what happens when nobody enforces it?

```{note}
The `decision_role` field is doing the heaviest lifting on this card. "Advisory only" is an ethical claim that must be true **in practice**, not just on paper. If nurses are evaluated on how often they agree with the flag, or if overriding it requires written justification while accepting it does not, the system is making the decision regardless of what the documentation says. **Look at the incentives, not the label.**
```

## 💡 Example: One Case, Four Frameworks

*A university deploys a model predicting which students will fail a required course, so advisors can intervene early.*

- **Consequentialist:** if intervention raises pass rates, deploy it. Measure the effect.
- **Deontological:** students have a right not to be prejudged. Does a prediction become self-fulfilling through advisor expectation?
- **Justice:** the model trained on historical outcomes. Does it flag first-generation students disproportionately because they historically had less support — and then attribute that to the students?
- **Care:** what does a struggling student actually need? Is an early conversation supportive or stigmatizing? Does it matter whether they know they were flagged?

None of these is the "right" answer. **A responsible deployment engages all four and documents which considerations governed.**

## 🧭 Reflection

> If fairness definitions conflict (Unit 10) and ethical frameworks conflict (Unit 11), is "responsible AI" achievable, or only ever a defensible position?
>
> Filling out a model card takes an hour. Why do so many deployed systems lack one?

**Connecting to HW11 (Ethical AI — Position Paper, 1 page, due Nov 23):** take a defensible position on a contested question and argue it. State which framework you are reasoning from and address the strongest objection to your view. **A one-page paper that engages one objection seriously beats one that surveys every principle.**

## 📘 Further Reading

- Dendritic Institute (2025). _AI Literacy Series — Appendix: Ethics and Responsible AI._
- Mitchell, M. et al. (2019). Model Cards for Model Reporting. _FAT* '19._
- Russell, S., & Norvig, P. (2022). _AI: A Modern Approach_, 4th Ed., Ch. 27. Pearson.
- Vallor, S. (2016). _Technology and the Virtues._ Oxford University Press.
