# Unit 6: AI Applications Across Domains

> **Session 10** · Sep 21 · **HW6 assigned Sep 21, due Sep 28**

This is a single session, and it is a hinge. Behind you are five units on how these systems work. Ahead of you are ten units on using them — in the office, on data, in code. This session connects the two.

The organizing claim is simple and worth stating plainly: **AI applications look wildly different across domains and are, underneath, a very short list of the same tasks.** Once you can see the task through the application, an unfamiliar AI product stops being a mystery and becomes a known quantity with known failure modes.

## Learning Objectives

After completing this unit, you will be able to:

- Reduce any AI application to one of a small set of **underlying task types**.
- Explain why the same task type carries different risk in different domains.
- Identify what a deployed system's **error costs** are, and who bears them.
- Assess whether a proposed AI application is solving a problem worth solving.

## Part I — The Short List Underneath

Almost every AI product you will meet is one of these, or a chain of them.

| Task type | What it does | Seen in |
| :--- | :--- | :--- |
| **Classification** | Assign a category | Spam filters, triage, fraud flags, content moderation |
| **Regression** | Predict a number | Demand forecasts, risk scores, delivery estimates |
| **Clustering** | Find groups | Customer segments, log anomaly detection |
| **Ranking** | Order a list | Search, feeds, recommendations |
| **Generation** | Produce content | Chat assistants, code completion, image tools |
| **Extraction** | Pull structure from mess | Invoice parsing, résumé screening, medical coding |
| **Control** | Choose actions over time | Routing, scheduling, robotics, autoscaling |

You have already built the first three yourself. The remaining four are the ones you will be *using* rather than building for the rest of this course.

```{note}
This table is a professional tool, not a taxonomy to memorize. When a vendor demonstrates something impressive, the useful question is not "how does it work?" but **"which row is this?"** — because the row tells you what its errors will look like and what evaluation you should be asking to see.
```

### 1.1 The Same Task, Different Stakes

Consider binary classification — the simplest thing on the list — deployed in four places.

| Application | A false positive means… | A false negative means… |
| :--- | :--- | :--- |
| Spam filter | A real email lands in junk | Junk lands in the inbox |
| Cancer screening | An unnecessary biopsy, and fear | A missed tumor |
| Loan approval | A safe borrower is denied credit | A default |
| Code vulnerability scanner | An engineer wastes an hour | A shipped exploit |

Identical mathematics. Radically different consequences. The spam filter can tolerate errors in either direction; the screening system cannot, and the *asymmetry* between its two error types is the entire design problem.

This is why Unit 3 insisted on the confusion matrix. A single accuracy number erases the distinction that matters most in every row of this table.

### 1.2 Where AI Actually Landed

A realistic map of deployment, as of the mid-2020s:

- **Healthcare** — imaging support (detection, not diagnosis), documentation, drug candidate screening. Adoption is slowest where liability is highest.
- **Finance** — fraud detection, credit scoring, algorithmic trading, compliance monitoring. Heavily regulated, and increasingly required to be explainable.
- **Software engineering** — code completion, test generation, log analysis, review assistance. The domain with the fastest adoption and the least established practice.
- **Education** — tutoring, feedback, content generation, and an unresolved argument about assessment.
- **Creative industries** — drafting, iteration, asset generation, alongside live litigation over training data.
- **Logistics and operations** — forecasting, routing, scheduling, maintenance prediction. Unglamorous, and where much of the real economic value has accumulated.

```{warning}
Notice the pattern. AI concentrates where errors are **cheap, detectable, and reversible**, and struggles to gain ground where they are none of those. That constraint is not a temporary limitation of current models. It is a permanent feature of deploying systems whose confidence is uncorrelated with their correctness — which is exactly what Unit 5 established.
```

### 1.3 Software Engineering as a Domain

Most of you are in this domain, so it deserves specific treatment. The applications you will encounter:

- **Code generation and completion** — generation, with correctness that is checkable by compilation and tests. This checkability is the reason adoption moved so fast here relative to medicine or law.
- **Code review assistance** — classification plus generation. Flags issues, drafts comments.
- **Test generation** — generation, evaluated by coverage and mutation score.
- **Log and incident analysis** — clustering and anomaly detection over high-volume, unlabeled data.
- **Documentation** — generation and extraction, which you will practice directly in Unit 10.

The professionally interesting fact is that this domain has an unusually good verification story — compilers, type checkers, test suites, and linters are all truth oracles that other fields lack — and has still produced a serious problem: **review burden**. Generated code arrives faster than humans can meaningfully read it. The bottleneck moved from writing to reviewing, and the reviewing was never the part that scaled.

## 💡 Example: Reading Four Products

For each, identify the task type, then the error asymmetry.

1. A tool that reads support tickets and routes them to the right team.
2. A tool that scans a repository and reports likely security vulnerabilities.
3. A tool that suggests the next line while you type code.
4. A tool that reads a résumé and predicts whether a candidate will be hired.

**Discussion.** (1) Classification; misroutes cost delay, symmetric and cheap. (2) Classification with brutal asymmetry — a missed vulnerability is catastrophic, a false alarm costs an hour, so it should be tuned to over-report and everyone will complain about the noise. (3) Generation; errors are caught by the compiler and the developer, cheap and detectable, which is why it works. (4) Classification, and the one to refuse — the "label" is a record of past human hiring decisions, so the model learns to reproduce whatever bias produced them. Unit 16 returns to this.

The fourth case is the one to sit with. Technically it is the *same* problem as the first. What differs is that the labels encode a contested human judgment rather than a fact about the world.

## 🧭 Reflection

> Pick an AI product you have personally used in the last month. Which row of the task table is it? What do its errors look like, and who pays for them — you, or someone else?
>
> If the answer is "someone else," what would change about the product's design if the cost landed on the people who built it?

**Connecting to HW6 (AI Application Analysis — Choose a Domain):** select one deployed AI application in a domain that interests you. Identify its task type, map its two error modes and their costs, and state who bears each. Then argue whether the application is well-matched to its domain — and if not, what would have to be true for it to become so.

**Looking ahead.** Part III moves from analysis to use. Starting next session you will be operating these systems rather than describing them, and everything from Unit 5's Part II applies to every tool you touch.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 1, 27. Pearson.
- Mitchell, M. (2020). _Artificial Intelligence: A Guide for Thinking Humans_, Part V. Penguin Books.
- Brynjolfsson, E., & McAfee, A. (2017). _Machine, Platform, Crowd_. W. W. Norton.
- Dendritic Institute (2025). _AI Literacy Series — Module 2: AI as Your Daily Assistant._
