# Unit 1: History and Evolution of AI

> **Sessions 1–4** · Aug 17, 19, 24, 26 · **HW1 assigned Aug 26, due Sep 02**

Artificial Intelligence is often described as a recent breakthrough. It is not. The field is roughly eighty years old, and almost every idea making headlines today has intellectual roots that stretch back decades. Understanding that history is not trivia, it is the fastest way to develop **judgment** about what is genuinely new and what is a familiar idea wearing new clothes.

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
- **1956** — The **Dartmouth Summer Research Project** coins the term *artificial intelligence*. The proposal famously suggested that significant progress could be made in a short period of time by few researchers.
- **1942** — Asimov's Three Laws of Robotics.** In the short story *Runaround*, Isaac Asimov proposed three rules that a robot must follow, in strict priority order: don't harm a human (or allow harm through inaction), obey human orders unless doing so violates the first law, and protect itself unless doing so violates the first two.  

### 1.2 The Symbolic Era (1956 – 1974)

Early AI was built on a clear and plausible approach: intelligence is **symbol manipulation**. If you can encode facts and rules precisely enough, reasoning follows.

- Programs proved mathematical theorems and played competent chess.
- **ELIZA** (1966) simulated a psychotherapist using pattern substitution, and unsettled its creator by how readily people confided in it.
- Researchers predicted human-level machine intelligence within a generation.

**What went wrong:** the approach worked beautifully on **toy problems** and collapsed on real ones. Encoding common-sense knowledge by hand turned out to require an unbounded number of rules and showed no generalization capability.

### 1.3 The First AI Winter (1974 – 1980)

Funding evaporated when results failed to match promises. The 1973 **Lighthill Report** in the UK concluded that AI had failed to deliver on its central claims, and government support was withdrawn on both sides of the Atlantic.

### 1.4 Expert Systems and the Second Winter (1980 – 1993)

**Expert systems** encoded specialist knowledge as explicit `IF–THEN` rules and found genuine commercial success. But they were brittle, expensive to maintain, and unable to learn. When the specialized hardware market collapsed in the late 1980s, so did the funding and a **second winter** followed.

> Both winters share a structure: **a real capability was oversold, the gap was noticed, and the correction overshot.** At that time, some key components to effective AI systems were missing: connectivity, data, storage and processing power, and algorithmic development.

## Part II — The Statistical Turn

### 2.1 Learning Instead of Programming

The decisive shift was philosophical before it was technical. Rather than *telling* a machine the rules, researchers began letting it **infer the rules from input (data)**.

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
Notice how much of the recent acceleration comes from **scale** (more data, more compute, larger models) and model developments. Whether that trend continues is one of the open questions of Unit 13.
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
    (2012, "AlexNet / DL"),
    (2016, "AlphaGo beats Lee Sedol"),
    (2017, "Transformers"),
    (2022, "GenAI"),
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
ax.set_title("Eighty Years of Artificial Intelligence", fontsize=14, weight="bold")
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
    "Free pizza for the hackathon winner — details in the break room",
]

print("SYMBOLIC APPROACH")
for t in tests:
    print(f"  spam={is_spam_rules(t)!s:<6} | {t[:56]}")
```

Run it. Then look carefully at the third message: *"Free pizza for the hackathon winner — details in the break room."* It contains two spam words but is obviously legitimate. Fixing that requires another rule. And that rule will break something else.

**This is exactly the wall that ended the symbolic era.** In Unit 3, you will solve the same problem by learning from examples instead, and you will see it handle cases nobody wrote a rule for.

---

## Part III — The Many Faces of AI

If you have spent any time reading about AI, you have probably noticed something confusing: the field seems to have a dozen names: Artificial intelligence; Machine learning; Deep learning; Data science; Generative AI; etc. Are these the same thing? Different things? Marketing?

The honest answer is: they are overlapping ideas, each named at a specific moment, by specific people, for a specific reason. Once you know *when* and *why* a term was coined, the confusion largely disappears. Terminology in this field is archaeology, each layer tells you what problem people were stuck on at the time.

This section walks through seven of these terms in roughly the order they entered the vocabulary.

### 3.1 Artificial Neural Networks (ANN)

**Proposed:** 1943 (foundational model) · 1958 (first learning machine) · 1986 (revival)

**The short version:** computing inspired by the wiring of the brain.

In 1943, neurophysiologist Warren McCulloch and logician Walter Pitts published a model of an artificial neuron, a tiny unit that adds up its inputs and fires if the total crosses a threshold. It was a mathematical abstraction, not a working machine, but it established a durable idea: *thinking might be reducible to networks of simple units*.

In 1958, Frank Rosenblatt built the **Perceptron**, a physical machine that could adjust its own connection strengths to learn a task. This is the first system most historians would call a learning machine.

**Why the term exists:** to distinguish brain-inspired, *learn-the-rules* computing from the logic-and-rules approach that dominated early AI. These were rival camps, not collaborators.

```{admonition} Two winters, one idea
:class: note
ANNs have died and returned twice. Minsky and Papert's 1969 book *Perceptrons* showed what single-layer networks could not do, and funding collapsed.
The field revived in 1986 when Rumelhart, Hinton, and Williams popularized **backpropagation**, a method for training networks with multiple layers.
(Paul Werbos had described the essential technique in his 1974 dissertation; it went largely unnoticed. Credit here is genuinely contested.)
```

### 3.2 Soft Computing

**Proposed:** early 1990s, by Lotfi Zadeh

**The short version:** a deliberate tolerance for imprecision, in exchange for solutions that actually work.

Zadeh had introduced **fuzzy logic** in 1965 as a way of representing "somewhat hot" rather than forcing a binary hot/cold. By the early 1990s he grouped fuzzy logic,
neural networks, and evolutionary computation under a single banner: **soft computing**, explicitly opposed to "hard" computing's demand for exactness and certainty.

**Why the term exists:** as an argument. Zadeh's claim was that real-world problems are messy, and that insisting on precise answers to imprecise questions is a way of guaranteeing failure. 
Accepting approximate, partially true answers is not sloppiness, it is what makes hard problems tractable.

You hear "soft computing" less today, but the argument won. Every modern AI system outputs probabilities and confidence scores rather than certainties.

### 3.3 Natural Computing

**Proposed:** consolidated as an umbrella term through the 1990s and 2000s

**The short version:** computing inspired by nature, computing *with* nature, and computing that simulates nature.

Rather than one invention, this is a family that accumulated over decades:

| Approach | Inspired by | Roughly |
|---|---|---|
| Evolutionary computation | Natural selection | 1960s–1975 |
| Artificial neural networks | The brain | 1943 onward |
| Swarm intelligence | Ant colonies, bird flocks | 1989–1995 |
| Artificial immune systems | The immune response | 1990s |
| DNA computing | Molecular biology | 1994 |

**Why the term exists:** to name a shared *strategy* rather than a shared technique.
The insight is that nature has already solved extraordinarily hard optimization and adaptation problems (evolution, immunity, collective foraging) and that these
solutions can be borrowed as algorithms. It is a research philosophy: when stuck,
look at how a living system handles the same pressure.

```{admonition} Worth noticing
:class: tip
Natural computing is one of the few terms here that is genuinely *broader* than AI rather than narrower.
Some of it (DNA computing) is not about intelligence at all, it is about using molecules as hardware.
```

### 3.4 Machine Learning

**Proposed:** 1959, by Arthur Samuel

**The short version:** programs that improve with experience instead of being told exactly what to do.

Samuel, working at IBM, wrote a checkers program that played games against itself and got better. 
In his 1959 paper he described this as giving computers "the ability to learn without being explicitly programmed", the phrase that defined the field.

**Why the term exists:** to name the alternative to hand-written rules. Recall the spam filter from the previous section: every rule you add may break something else. 
ML is the response to that wall. Instead of writing rules, you show the system thousands of labeled examples and let it derive the rules itself.

For roughly thirty years this remained a niche within AI. It became *the* mainstream of AI in the 1990s and 2000s, for two unglamorous reasons: data became abundant, and computers became faster.

### 3.5 Data Mining

**Proposed:** as a pejorative in the 1960s–70s · reclaimed 1989 (first KDD workshop) · mainstream through the 1990s

**The short version:** finding useful patterns in large databases, especially patterns nobody thought to look for.

The term began as an accusation. Statisticians used "data mining", along with "data dredging" and "fishing expedition", to describe the sin of searching a dataset until something looked significant, then reporting it as though you had predicted it in advance. Search hard enough and any dataset will yield a coincidence.

In 1989, Gregory Piatetsky-Shapiro organized the first workshop on **Knowledge Discovery in Databases (KDD)**, and the community adopted the insult as its name.
The reasoning was that the sin is not looking, it is looking without validating what you find. Done with proper holdout data and significance correction, pattern discovery is legitimate and enormously valuable.

**Why the term exists:** because businesses had accumulated databases far larger than anyone could inspect by hand, and the interesting patterns were unknown in advance. 
Machine learning as practiced then mostly answered a question you posed; data mining asked *what is in here that I did not think to ask about?*

```{admonition} Why you hear it less now
:class: note
Data mining did not fail, it was absorbed. Its techniques live on inside machine learning and data science, but is still one of the most widely used process models for data projects. The term faded partly because "mining" acquired an unfortunate connotation once consumer data collection became a public concern.
```

```{admonition} The original sin, still worth avoiding
:class: warning
The pejorative meaning has not gone away. If you test enough hypotheses against one dataset, you will find "significant" results by chance alone.
This has a name — **p-hacking** — and it is a live problem across the sciences. When your capstone finds a striking pattern, the first question to ask is whether you went looking for that specific pattern, or whether you found it by looking at everything.
```

### 3.6 Data Science

**Proposed:** 1974 (first use) · 2001 (as a discipline) · 2008 (as a job title)

**The short version:** the whole pipeline around the data, not just the algorithm.

The term appears as early as 1974 in Peter Naur's writing. It was proposed as a discipline in 2001 by statistician William Cleveland, who argued that statistics should expand to include computing and real data problems. It became a *career* in 2008, when Jeff Hammerbacher (Facebook) and DJ Patil (LinkedIn) adopted "data scientist" as a job title for people who did not fit existing categories.

**Why the term exists:** because the algorithm is the easy part. Somebody has to find the data, clean it, ask whether it is representative, check what it is missing, build the pipeline, and explain the result to a decision-maker who does not care about gradient descent. That work needed a name.

```{admonition} A distinction that matters for your capstone
:class: important
Machine learning is a *method*. Data science is a *process* that often uses that method.
A data science project can succeed with no ML at all, sometimes the answer is a well-constructed chart and an honest caveat.
```

### 3.7 Deep Learning

**Proposed:** term used in this sense from ~2000–2006 · mainstream from 2012

**The short version:** neural networks with many layers, made practical.

The word "deep" refers simply to the number of layers between input and output. Rina Dechter used "deep learning" in a different machine-learning context in 1986; Igor Aizenberg applied it to neural networks around 2000; Geoffrey Hinton and colleagues popularized it from 2006 with methods for training deep networks that previously would not train.

The moment everything changed was 2012, when a deep network called **AlexNet** won the ImageNet image-recognition competition by a margin so wide it ended the debate.
Within two years, essentially the whole field had switched.

**Why the term exists:** partly as rebranding. "Neural networks" carried the stigma of two failed hype cycles. But it also names something real: depth is what lets a network build up concepts in stages, from edges to shapes to objects, without anyone specifying those stages.

```{admonition} What actually changed in 2012
:class: note
Not the theory. The core ideas were decades old. What changed was **GPUs** (graphics chips repurposed for the math) and **ImageNet** (a labeled dataset of 14 million images).
Old ideas plus new data plus new hardware. Keep this pattern in mind, as it recurs.
```

### 3.8 Generative AI

**Proposed:** techniques from 2013–2017 · the term entered common use in 2022–2023

**The short version:** systems that produce new content rather than simply performing inferences over existing content.

Most AI before this was *discriminative*: is this spam or not, is this a tumor or not, which of these ten digits is this. 
Generative systems produce the artifact itself, such as text, images, audio, and code.

Key milestones:

- **2013** — variational autoencoders
- **2014** — Generative Adversarial Networks (Ian Goodfellow), which pit two networks against each other, one generating and one detecting fakes
- **2017** — the **Transformer** architecture ("Attention Is All You Need"), the foundation of every major language model since
- **2022** — ChatGPT launches in November and reaches an estimated 100 million users within two months

**Why the term exists:** because the shift is real, not cosmetic. When a system infers (e.g. classifies), you can check whether it was right. 
When a system *generates*, "right" is often undefined, which is why questions of authorship, accuracy, and accountability suddenly became urgent for everyone rather than for specialists.

Note the five-year gap between the Transformer paper and ChatGPT. The technology was public and published the entire time.

### 📅 Putting It Together

| Term | Anchor year | Named in response to |
|---|---|---|
| Artificial neural networks | 1943 / 1958 | Can we compute the way brains do? |
| Fuzzy logic → soft computing | 1965 / early 1990s | Exact answers to inexact questions are useless |
| Natural computing | 1990s–2000s | Nature already solved hard problems |
| Machine learning | 1959 | Hand-written rules do not scale |
| Data science | 2001 / 2008 | The algorithm is the easy part |
| Deep learning | ~2006 / 2012 | Depth works, given data and hardware |
| Generative AI | 2014–2017 / 2022 | Systems that create, not just classify |

```{admonition} How they nest — approximately
:class: warning
You will often see a tidy diagram: AI contains ML contains deep learning contains generative AI.
It is a useful first approximation and it is not quite right.
Natural computing overlaps AI without being contained by it.
Data science overlaps ML but extends well outside it.
Plenty of generative techniques are not deep learning.
Treat the nesting as a rough map, not a taxonomy.
```

## ⚙️ Hands-On: Mapping the Vocabulary

The first hands-on plotted *events*. This one plots *names* — when each term in Part III entered the vocabulary, and what argument it was making at the time.

**Copy this into Colab (or any Python environment) and run it.**

```python
import matplotlib.pyplot as plt

# --- When each term entered the vocabulary, and what it was named against ---
terms = [
    (1943, "Artificial Neural Networks",  "Can we compute the way brains do?"),
    (1959, "Machine Learning",            "Hand-written rules do not scale"),
    (1965, "Fuzzy Logic / Soft Computing","Exact answers to vague questions"),
    (1977, "Exploratory Data Analysis",   "Look at the data before modelling"),
    (1989, "Data Mining",                 "What is in here I did not ask about?"),
    (1990, "Natural Computing",           "Nature already solved hard problems"),
    (2001, "Data Science",                "The algorithm is the easy part"),
    (2006, "Deep Learning",               "Depth works, given data and hardware"),
    (2014, "Generative AI",               "Systems that create, not just sort"),
]

winters = [(1974, 1980, "First AI Winter"), (1987, 1993, "Second AI Winter")]
TODAY = 2026

fig, ax = plt.subplots(figsize=(13, 6))

# Shade the AI winters, exactly as in the first hands-on
for start, end, label in winters:
    ax.axvspan(start, end, color="lightsteelblue", alpha=0.45, zorder=0)
    ax.text((start + end) / 2, len(terms) - 0.3, label, ha="center",
            fontsize=8.5, style="italic", color="navy")

# One bar per term, running from the year it was coined to today
for i, (year, name, _) in enumerate(terms):
    row = len(terms) - 1 - i
    ax.barh(row, TODAY - year, left=year, height=0.55,
            color="darkgreen", alpha=0.30, zorder=2)
    ax.plot(year, row, "o", color="darkgreen", markersize=9, zorder=3)
    ax.text(year - 1.5, row, str(year), ha="right", va="center",
            fontsize=9, weight="bold")
    ax.text(year + 2, row + 0.02, name, ha="left", va="center", fontsize=9.5)

ax.set_yticks([])
ax.set_xlim(1935, TODAY + 4)
ax.set_ylim(-0.8, len(terms) + 0.2)
ax.set_xlabel("Year")
ax.set_title("The Many Faces of AI: when each name entered the vocabulary",
             fontsize=14, weight="bold")
ax.spines[["left", "right", "top"]].set_visible(False)
plt.tight_layout()
plt.show()

print("Nothing on this chart has ended. Every bar runs to today.\n")
for year, name, against in terms:
    print(f"  {year}  {name:<30} {against}")
```

Look at the shape the bars make: a staircase, each step starting later than the last and none of them stopping. **The terms accumulated; they did not replace one another.** Machine learning did not retire neural networks, and generative AI has not retired data mining. All nine are in active use today, which is precisely why the vocabulary feels so crowded when you first encounter it.

Then notice what crosses the shaded bands. Artificial neural networks, machine learning, and fuzzy logic were all named *before* the first AI winter and were all still being worked on during both. Winters are periods when funding and attention collapse, not periods when the ideas disappear.

**Try changing it:**

1. The date for Natural Computing is the softest on this chart — its components arrived separately (evolutionary computation in the 1960s, swarm intelligence around 1990, DNA computing in 1994). Pick the date you think is most defensible, change it, and be ready to say why. Which other rows could reasonably move?
2. Add a row for **Symbolic AI**, which Part I covered but Part III did not list. What year do you give it, and does its bar run to today like the others, or does it stop?
3. Three terms sit close together between 1977 and 2001 — EDA, data mining, and data science. What was happening in the world during those years that would cause three names for working with data to appear in such quick succession?

## 🧭 Reflection

> Every era of AI believed it had found the right approach, and every era was partly right and partly wrong.
> Which of today's assumptions do you think will look naive in twenty years?
>
> The Dartmouth proposal estimated that a summer would suffice. What makes intelligence so much harder to build than its pioneers expected?

**Connecting to HW1 (Reflection: AI in Your Daily Life, due Sep 02):** as you inventory the AI systems you interact with daily, try to identify for each whether it is closer to the **rule-based** or the **learning-based** tradition. Some are genuinely hybrids.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 1. Pearson.
- Mitchell, M. (2020). _Artificial Intelligence: A Guide for Thinking Humans_, Part I. Penguin Books.
- de Castro, L. N. (2026). _Exploratory Data Analysis: Descriptive Statistics, Visualization, and Dashboard Design_, CRC Press.
- Turing, A. M. (1950). Computing Machinery and Intelligence. _Mind_, 59(236), 433–460.
- McCulloch, W. & Pitts, W. (1943). *A Logical Calculus of the Ideas Immanent in Nervous Activity.*
- Samuel, A. (1959). *Some Studies in Machine Learning Using the Game of Checkers.*
- Zadeh, L. (1965). *Fuzzy Sets.*
- Rumelhart, D., Hinton, G. & Williams, R. (1986). *Learning representations by back-propagating errors.*
- Cleveland, W. (2001). *Data Science: An Action Plan for Expanding the Technical Areas of the Field of Statistics.*
- de Castro, L. N. (2006). *Fundamentals of Natural Computing.* Chapman & Hall/CRC.
- Goodfellow, I. et al. (2014). *Generative Adversarial Networks.*
- Vaswani, A. et al. (2017). *Attention Is All You Need.*
- Mitchell, M. (2019). *Artificial Intelligence: A Guide for Thinking Humans.*
- Dendritic Institute (2025). _AI Literacy Series — Module 1: A Brief History of AI._
