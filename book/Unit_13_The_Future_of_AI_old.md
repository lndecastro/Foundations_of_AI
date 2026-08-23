# Unit 13: The Future of AI

> **Session 29** · TBD · *Final unit before Capstone Presentations on Dec 02*

This unit is deliberately the least certain in the book. Its purpose is not to predict — it is to give you a **framework for evaluating predictions**, including the confident ones you will encounter throughout your career.

## Learning Objectives

After completing this unit, you will be able to:

- Distinguish **trend extrapolation** from **forecasting** and explain why the difference matters.
- Summarize the principal open questions about AI's trajectory.
- Evaluate claims about AI's future using evidence rather than intuition or authority.
- Identify which of your own skills are likely to remain durable.

## Part I — What We Can Actually Observe

Rather than speculating, start with measurable trends.

### 1.1 The Scaling Hypothesis

The dominant driver of recent progress has been **scale**: more parameters, more data, more compute. Empirically, performance has improved predictably as these grow.

The open question — genuinely open, disputed by serious researchers — is whether this continues:

| Position | Claim | Evidence cited |
| :--- | :--- | :--- |
| **Scaling continues** | Capabilities keep emerging with scale | Consistent historical gains; emergent abilities |
| **Scaling plateaus** | Returns diminish; data and energy limit growth | Rising costs; finite high-quality text |
| **New ideas needed** | Current architectures cannot reason or model causality | Persistent failures on reasoning, brittleness (Unit 5) |

```{important}
Note that **all three positions are held by credentialed researchers with access to the same evidence.** When experts with identical data disagree this sharply, the honest conclusion is that the question is unresolved — not that one side is obviously right and you simply need to identify which.
```

## ⚙️ Hands-On: The Perils of Extrapolation

Trend extrapolation is the most common form of technology forecasting and the most commonly abused. This activity shows you why — by fitting three plausible curves to the same data and watching them diverge wildly.

```python
import numpy as np, matplotlib.pyplot as plt
from scipy.optimize import curve_fit

# Illustrative: relative capability on a benchmark over time
years = np.array([2012, 2014, 2016, 2018, 2020, 2022, 2024])
capability = np.array([1.0, 2.1, 4.5, 9.0, 19.5, 41.0, 78.0])

t = years - 2012
future = np.linspace(0, 24, 200)          # project to 2036

# Three models, all fitting the SAME data well
def exponential(x, a, b):  return a * np.exp(b * x)
def logistic(x, L, k, x0): return L / (1 + np.exp(-k * (x - x0)))
def power_law(x, a, b):    return a * (x + 1) ** b

fits = {}
fits["Exponential"] = curve_fit(exponential, t, capability, p0=[1, 0.2])[0]
fits["Logistic (S-curve)"] = curve_fit(
    logistic, t, capability, p0=[100, 0.4, 12], maxfev=20000)[0]
fits["Power law"] = curve_fit(power_law, t, capability, p0=[1, 2])[0]

funcs = {"Exponential": exponential, "Logistic (S-curve)": logistic,
         "Power law": power_law}

plt.figure(figsize=(11, 6))
plt.scatter(t, capability, s=90, color="black", zorder=5,
            label="observed data (2012-2024)")

for name, params in fits.items():
    y = funcs[name](future, *params)
    resid = np.sum((funcs[name](t, *params) - capability) ** 2)
    plt.plot(future, y, lw=2.5, label=f"{name} (fit error {resid:.1f})")
    print(f"{name:20s} 2036 projection: {funcs[name](24, *params):>12,.0f}")

plt.axvline(12, color="gray", ls="--")
plt.text(12.3, 10, "today", fontsize=10, color="gray")
plt.xticks(range(0, 25, 4), [str(2012 + i) for i in range(0, 25, 4)])
plt.yscale("log")
plt.ylabel("Relative capability (log scale)")
plt.title("Three curves. Same data. Wildly different futures.",
          fontsize=13, weight="bold")
plt.legend(); plt.tight_layout(); plt.show()
```

Run it and read the printed projections. All three curves fit the historical data closely — yet they predict roughly **292**, **857**, and **4,863** for 2036. A **seventeen-fold spread**, from three models that all look correct against everything observed so far.

Worse, the *best-fitting* curve is the logistic — the one predicting a plateau. Fit quality does not settle the question either.

```{warning}
**This is the central lesson of the unit.** You cannot distinguish an exponential from the early portion of an S-curve using only the early portion. Every technology that eventually plateaued looked exponential on the way up — including, notably, the ones people were most confident about.

So when you encounter a confident forecast about AI, ask: *what curve is being assumed, and what evidence rules out the alternatives?* Usually the answer is that nothing rules them out. The forecast is a choice of curve dressed as an observation.
```

**Try changing it:**

1. Remove the last two data points and refit. How much do the projections change? **Forecasts built on short histories are extremely unstable.**
2. Add a hypothetical 2026 point at 95 (a slowdown). Which model adapts most?
3. Find a real forecast about AI from 2015. How did it do? What curve was it assuming?

## Part II — Open Questions

### 2.1 Capability Questions

- **Can current architectures reason, or only pattern-match very well?** Unit 5's failure demonstration is one data point on this.
- **Will causal understanding emerge from scale, or does it require new methods?**
- **What are the physical limits?** Energy, data availability, and chip supply are real constraints.

### 2.2 Economic Questions

- Which tasks get automated, which get augmented, and which are untouched?
- Does AI compress or widen wage inequality?
- Historically, technology has created more jobs than it destroyed — **but never on a guarantee, and never without painful transitions for specific groups.**

### 2.3 Societal Questions

- What happens to shared reality when synthetic media is indistinguishable from recorded media?
- Does AI concentrate power in the few organizations that can afford to train frontier models?
- What is lost when we delegate cognitive work — and what was lost when we delegated arithmetic to calculators?

### 2.4 Safety Questions

- How do we verify systems whose behavior we cannot fully characterize?
- Can we specify objectives that do not produce unintended optimization? (Unit 2's performance-measure problem, scaled up.)
- What governance would be needed for systems substantially more capable than today's?

## Part III — What Remains Durable

A practical closing question: what should you invest in learning?

| Likely to be automated | Likely to remain valuable |
| :--- | :--- |
| Routine information retrieval | Deciding **which question** to ask |
| First-draft generation | Judging whether a draft is **right** |
| Pattern recognition in clean data | Recognizing when the data is **wrong** |
| Standardized analysis | Framing an **ambiguous, novel problem** |
| Summarization | Deciding what **matters** and to whom |

> The pattern: **AI is strong at execution within a well-specified frame and weak at choosing the frame.** Every unit of this course reinforced this. The spam filter could not decide that "free coffee" was benign. K-Means found groups but could not name them. The digit classifier could not notice it had left familiar territory. The hiring model could not know that its labels encoded past discrimination.

## 🧭 Reflection

> You have spent a semester learning how these systems work and where they fail. Has your estimate of their near-term impact gone up or down?
>
> The 1956 Dartmouth proposal estimated that a summer would suffice for major progress. Which of today's confident predictions will read the same way in 2056?
>
> **What would change your mind?** If you cannot answer that about your own view of AI's future, the view is not yet evidence-based.

## A Closing Note

You began this course with no assumption of technical background. You have now built a classifier, trained a neural network, made one fail, produced a discriminatory hiring model without giving it gender, re-identified people in "anonymous" data, and watched three equally valid curves predict three different futures.

That is not a survey of AI. **That is a working relationship with it** — and it is a far better foundation for the next forty years of your career than familiarity with any particular tool would be.

The systems will keep changing. The questions you now know to ask will not.

## 📘 Further Reading

- Mitchell, M. (2020). _Artificial Intelligence: A Guide for Thinking Humans_, Ch. 14–16. Penguin Books.
- Russell, S. (2019). _Human Compatible: Artificial Intelligence and the Problem of Control._ Viking.
- Russell, S., & Norvig, P. (2022). _AI: A Modern Approach_, 4th Ed., Ch. 28. Pearson.
- Brynjolfsson, E., & McAfee, A. (2014). _The Second Machine Age._ W. W. Norton.
