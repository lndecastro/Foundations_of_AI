# Unit 1: History and Evolution of AI

> **Sessions 1–4** · Aug 17, 19, 24, 26 · **HW1 assigned Aug 26, due Sep 02**

Artificial Intelligence is often described as a recent breakthrough. It is not. The field is roughly seventy years old, and almost every idea making headlines today has intellectual roots that stretch back decades. Understanding that history is not trivia — it is the fastest way to develop **judgment** about what is genuinely new and what is a familiar idea wearing new clothes.

## Learning Objectives

After completing this unit, you will be able to:

- Trace the **major eras** of AI development and identify what drove each transition.
- Explain the difference between **symbolic (rule-based)** and **statistical (learning-based)** approaches to AI.
- Describe what an **"AI winter"** was, why they happened, and what they teach us about hype cycles.
- Situate today's systems within a longer arc of technical development.

## Part I — The Origins

### 1.1 Before the Name

The idea of mechanical reasoning predates computers entirely. What changed in the mid-twentieth century was the arrival of machines that could actually execute it.

- **1943** — McCulloch and Pitts propose a mathematical model of a neuron. This is the ancestor of every neural network in use today.
- **1950** — Alan Turing publishes _Computing Machinery and Intelligence_, proposing what we now call the **Turing Test** and reframing "can machines think?" into a question about observable behavior.
- **1956** — The **Dartmouth Summer Research Project** coins the term *artificial intelligence*. The proposal famously suggested that significant progress could be made in a single summer by ten people.

> That last detail is worth sitting with. **Underestimating difficulty has been a constant in this field**, from 1956 to the present day.

### 1.2 The Symbolic Era (1956 – 1974)

Early AI was built on a clear and plausible bet: intelligence is **symbol manipulation**. If you can encode facts and rules precisely enough, reasoning follows.

- Programs proved mathematical theorems and played competent chess.
- **ELIZA** (1966) simulated a psychotherapist using pattern substitution — and unsettled its creator by how readily people confided in it.
- Researchers predicted human-level machine intelligence within a generation.

**What went wrong:** the approach worked beautifully on **toy problems** and collapsed on real ones. Encoding common-sense knowledge by hand turned out to require an unbounded number of rules.

### 1.3 The First AI Winter (1974 – 1980)

Funding evaporated when results failed to match promises. The 1973 **Lighthill Report** in the UK concluded that AI had failed to deliver on its central claims, and government support was withdrawn on both sides of the Atlantic.

### 1.4 Expert Systems and the Second Winter (1980 – 1993)

**Expert systems** encoded specialist knowledge as explicit `IF–THEN` rules and found genuine commercial success. But they were brittle, expensive to maintain, and unable to learn. When the specialized hardware market collapsed in the late 1980s, so did the funding — a **second winter** followed.

> Both winters share a structure: **a real capability was oversold, the gap was noticed, and the correction overshot.** Keep this pattern in mind as you read AI coverage today.

## Part II — The Statistical Turn

### 2.1 Learning Instead of Programming

The decisive shift was philosophical before it was technical. Rather than *telling* a machine the rules, researchers began letting it **infer the rules from data**.

| Era | Approach | How knowledge enters the system |
| :--- | :--- | :--- |
| **Symbolic** | Rules written by experts | Humans encode it explicitly |
| **Statistical** | Patterns learned from examples | The machine extracts it from data |

This is the single most important conceptual boundary in the course, and Unit 3 returns to it in detail.

### 2.2 Milestones of the Modern Era

- **1997** — **Deep Blue** defeats Garry Kasparov at chess, largely through brute-force search rather than learning.
- **2012** — **AlexNet** wins the ImageNet competition by a wide margin, triggering the deep learning boom. Three ingredients converged: large labeled datasets, GPU computing, and better training techniques.
- **2016** — **AlphaGo** defeats Lee Sedol at Go, combining deep networks with reinforcement learning.
- **2017** — The **Transformer** architecture is introduced, enabling the language models that follow.
- **2018–present** — Large language models scale rapidly; generative AI reaches general public use.

```{note}
Notice how much of the recent acceleration comes from **scale** — more data, more compute, larger models — rather than from fundamentally new mathematics. Whether that trend continues is one of the genuinely open questions of Unit 13.
```

## ⚙️ Hands-On: Plotting the Arc of AI

Let's make the history visible. This snippet builds a timeline of major milestones and shades the two AI winters.

**Copy this into Colab (or any Python environment) and run it.**

```python
import matplotlib.pyplot as plt

# --- Major milestones in AI history -------------------------------
milestones = [
    (1943, "McCulloch-Pitts neuron"),
    (1950, "Turing Test proposed"),
    (1956, "Dartmouth: 'AI' coined"),
    (1966, "ELIZA"),
    (1980, "Expert systems boom"),
    (1997, "Deep Blue beats Kasparov"),
    (2012, "AlexNet / deep learning"),
    (2016, "AlphaGo beats Lee Sedol"),
    (2017, "Transformer architecture"),
    (2022, "Generative AI goes mainstream"),
]

winters = [(1974, 1980, "First AI Winter"),
           (1987, 1993, "Second AI Winter")]

fig, ax = plt.subplots(figsize=(13, 5))

# Shade the AI winters
for start, end, label in winters:
    ax.axvspan(start, end, color="lightsteelblue", alpha=0.5)
    ax.text((start + end) / 2, -0.75, label, ha="center",
            fontsize=9, style="italic", color="navy")

# Plot milestones, alternating labels above and below the line
for i, (year, event) in enumerate(milestones):
    height = 0.6 if i % 2 == 0 else -0.6
    ax.plot([year, year], [0, height], color="gray", linewidth=1)
    ax.plot(year, 0, "o", color="darkgreen", markersize=8)
    ax.text(year, height * 1.15, f"{year}\n{event}", ha="center",
            va="bottom" if height > 0 else "top", fontsize=8.5)

ax.axhline(0, color="black", linewidth=1.5)
ax.set_xlim(1935, 2030)
ax.set_ylim(-1.6, 1.4)
ax.set_yticks([])
ax.set_xlabel("Year")
ax.set_title("Seventy Years of Artificial Intelligence", fontsize=14, weight="bold")
ax.spines[["left", "right", "top"]].set_visible(False)
plt.tight_layout()
plt.show()
```

**Try changing it:**

1. Add three milestones you think belong on this timeline. What did you add, and why?
2. Change the shaded regions. Do *you* think we are in a hype cycle right now? Where would you shade it?
3. Notice the visual gap between 1956 and 1997. What does that spacing tell you about the pace of the field?

## 💡 Example: Rules vs. Learning, Side by Side

The clearest way to feel the symbolic/statistical divide is to solve the same problem twice.

```python
# --- Approach 1: SYMBOLIC (1970s style) ---------------------------
# A human writes the rules explicitly.
def is_spam_rules(email):
    spam_words = ["free", "winner", "prize", "urgent", "click here"]
    text = email.lower()
    hits = sum(word in text for word in spam_words)
    return hits >= 2

tests = [
    "URGENT: you are a WINNER, click here for your free prize",
    "Hi Professor, attaching my homework for Unit 1. Thanks!",
    "Free coffee in the break room today",
]

print("SYMBOLIC APPROACH")
for t in tests:
    print(f"  spam={is_spam_rules(t)!s:<6} | {t[:55]}")
```

Run it. Then look carefully at the third message: *"Free coffee in the break room today."* It contains a spam word but is obviously legitimate. Fixing that requires another rule. And that rule will break something else.

**This is exactly the wall that ended the symbolic era.** In Unit 3, you will solve the same problem by learning from examples instead — and you will see it handle cases nobody wrote a rule for.

## 🧭 Reflection

> Every era of AI believed it had found the right approach, and every era was partly right and partly wrong.
> Which of today's assumptions do you think will look naive in twenty years?
>
> The Dartmouth proposal estimated that a summer would suffice. What makes intelligence so much harder to build than its pioneers expected?

**Connecting to HW1 (Reflection: AI in Your Daily Life, due Sep 02):** as you inventory the AI systems you interact with daily, try to identify for each whether it is closer to the **rule-based** or the **learning-based** tradition. Some are genuinely hybrids.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 1. Pearson.
- Mitchell, M. (2020). _Artificial Intelligence: A Guide for Thinking Humans_, Part I. Penguin Books.
- Turing, A. M. (1950). Computing Machinery and Intelligence. _Mind_, 59(236), 433–460.
- Dendritic Institute (2025). _AI Literacy Series — Module 1: A Brief History of AI._
