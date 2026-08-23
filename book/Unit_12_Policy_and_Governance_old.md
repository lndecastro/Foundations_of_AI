# Unit 12: Policy, Regulation and Governance

> **Sessions 27–28** · Nov 23 and TBD · **HW12 assigned Nov 23, due TBD**
> *(Note: Nov 25 falls in Thanksgiving Week — no class.)*

Ethics tells us what we ought to do. **Governance is the machinery that makes it happen** — or fails to. This unit surveys how governments and institutions are actually attempting to regulate AI, and asks the harder question of whether regulation can keep pace with the technology.

```{warning}
**This is the fastest-moving material in the course.** Regulatory frameworks described here may have changed since this book was written. Verify current status before relying on any specific provision — and treat that instability as itself a finding about governing emerging technology.
```

## Learning Objectives

After completing this unit, you will be able to:

- Compare the major **regulatory approaches** to AI and the philosophies behind them.
- Apply a **risk-tier framework** to classify AI systems.
- Explain the **pacing problem** and why it makes technology regulation structurally hard.
- Identify the roles of institutions beyond government in AI governance.

## Part I — Three Regulatory Philosophies

The major jurisdictions have taken meaningfully different approaches, and the differences are philosophical rather than merely technical.

| Approach | Exemplar | Logic | Trade-off |
| :--- | :--- | :--- | :--- |
| **Risk-based, prescriptive** | EU AI Act | Classify systems by risk; impose obligations proportionally | Clear rules; may burden low-risk innovation |
| **Sectoral, existing-agency** | United States | Apply existing regulators (FDA, FTC, EEOC) domain by domain | Flexible and expert; produces gaps and inconsistency |
| **State-directed** | China | Align AI development with state objectives; register algorithms | Rapid coordinated action; different value premises |

### 1.1 The EU Risk Tiers

The EU AI Act's structure has become the most widely referenced model, whatever one thinks of it.

| Tier | Treatment | Examples |
| :--- | :--- | :--- |
| **Unacceptable** | Prohibited | Social scoring by governments; certain manipulative systems |
| **High risk** | Strict obligations before and after deployment | Hiring, credit, medical devices, critical infrastructure, education access |
| **Limited risk** | Transparency obligations | Chatbots must disclose they are AI; synthetic media labeling |
| **Minimal risk** | No specific obligation | Spam filters, recommendation systems, game AI |

### 1.2 The US Sectoral Landscape

Rather than one AI law, the US applies existing authorities:

- **EEOC** — employment discrimination, including algorithmic hiring tools
- **FDA** — AI as a medical device
- **FTC** — unfair or deceptive practices, including exaggerated AI claims
- **CFPB** — credit decisions and adverse-action notice requirements
- **State laws** — Colorado, California, Illinois, Texas and others have enacted AI-specific provisions

> **The consequence:** in the US, an AI hiring tool is regulated as *employment discrimination*, not as *AI*. This has an underappreciated advantage — the underlying harm is what matters, not the technology used to cause it — and a real disadvantage in gaps and inconsistency across states.

### 1.3 Standards and Soft Governance

Not all governance is law:

- **NIST AI Risk Management Framework** (US) — voluntary, widely adopted
- **ISO/IEC 42001** — AI management system certification
- **OECD AI Principles** — intergovernmental, adopted by 40+ countries
- **UNESCO Recommendation on AI Ethics** — global normative instrument

## ⚙️ Hands-On: Classifying Systems by Risk Tier

Regulatory reasoning is a skill. This activity makes you apply the criteria rather than memorize them.

```python
import pandas as pd

# --- A rough EU-AI-Act-style classifier ---------------------------
def classify_risk(affects_fundamental_rights, safety_critical,
                  automated_decision, interacts_with_people,
                  government_scoring=False, manipulative=False):
    if government_scoring or manipulative:
        return "UNACCEPTABLE — prohibited"
    if affects_fundamental_rights or safety_critical:
        return "HIGH RISK — conformity assessment, documentation, oversight"
    if interacts_with_people:
        return "LIMITED RISK — transparency obligation"
    return "MINIMAL RISK — no specific obligation"

systems = [
    # name, rights, safety, automated, interacts, gov_score, manipulative
    ("Resume screening tool",          True,  False, True,  False, False, False),
    ("Spam filter",                    False, False, True,  False, False, False),
    ("Tumor detection in CT scans",    True,  True,  False, False, False, False),
    ("Customer service chatbot",       False, False, False, True,  False, False),
    ("Credit scoring model",           True,  False, True,  False, False, False),
    ("Music recommendation",           False, False, False, False, False, False),
    ("Autonomous vehicle perception",  True,  True,  True,  False, False, False),
    ("University admissions ranking",  True,  False, True,  False, False, False),
    ("Citizen social credit scoring",  True,  False, True,  False, True,  False),
]

rows = []
for name, *flags in systems:
    rows.append({"System": name, "Classification": classify_risk(*flags)})

pd.set_option("display.max_colwidth", 60)
print(pd.DataFrame(rows).to_string(index=False))
```

**Try changing it:**

1. **Add your capstone project** to the list. Which tier does it fall into? What obligations would follow? This belongs in your final presentation.
2. The classifier treats `affects_fundamental_rights` as a single boolean. Which of the nine systems was hardest to assign? **Ambiguity at the boundary is where regulation actually gets contested.**
3. A general-purpose language model can perform *every* task above. How should a system be classified when its risk depends entirely on how someone uses it? This is the hardest open problem in AI regulation.

## Part II — Why This Is Structurally Hard

### 2.1 The Pacing Problem

Technology develops faster than law. By the time a statute is drafted, debated, and enacted, the systems it describes have changed. This produces two failure modes:

- **Regulating too specifically** → rules become obsolete or trivially circumvented
- **Regulating too generally** → rules become unenforceable and provide no guidance

### 2.2 The Definitional Problem

Laws require definitions, and "artificial intelligence" resists definition. Too broad and you capture ordinary spreadsheets; too narrow and next year's architecture escapes.

### 2.3 The Jurisdictional Problem

A model trained in one country, hosted in a second, and used in a third — whose rules apply? The **Brussels Effect** describes how EU rules propagate globally because compliance is cheaper than maintaining separate systems, but it is not universal.

### 2.4 The Capacity Problem

Regulators must evaluate systems that their own staff often cannot inspect, using expertise concentrated in the companies being regulated. This asymmetry is not easily fixed by writing more rules.

## Part III — Governance Beyond Government

| Actor | Governance role |
| :--- | :--- |
| **Universities** | IRB review, research norms, training the next generation |
| **Professional bodies** | Codes of conduct (IEEE, ACM, medical boards) |
| **Standards organizations** | Technical specifications enabling compliance |
| **Companies** | Internal review boards, red-teaming, release policies |
| **Civil society** | Auditing, investigative journalism, litigation |
| **Insurers** | Liability pricing — an underrated and growing lever |

```{note}
That last row deserves attention. **Insurance may end up governing AI more effectively than legislation.** If insurers refuse to cover harms from undocumented models, organizations will document their models — faster than any statute could compel. Follow where liability lands.
```

## 🧭 Reflection

> The US regulates AI harms through existing law; the EU regulates AI as a distinct category. Which approach would you rather be subject to as a developer? As a citizen affected by a decision? **Notice if your answers differ.**
>
> If regulation always lags technology, what should fill the gap in the meantime — and who decides?

**Connecting to HW12 (AI Policy Brief — Summary & Response):** select one specific regulatory instrument, summarize what it actually requires, and take a position on whether it will achieve its aim. Precision about the actual provisions beats broad commentary.

## 📘 Further Reading

- U.S. GAO. _Artificial Intelligence: Report 25-107653._ (Available in course materials)
- NIST (2023). _AI Risk Management Framework 1.0._
- Bradford, A. (2020). _The Brussels Effect._ Oxford University Press.
- OECD. _AI Principles._ oecd.ai
- Russell, S., & Norvig, P. (2022). _AI: A Modern Approach_, 4th Ed., §27.3–27.4. Pearson.
