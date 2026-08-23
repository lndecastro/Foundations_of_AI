# Unit 7: Prompt Engineering

> **Sessions 18–19** · Oct 19, Oct 21 · **HW7 assigned Oct 21, due Oct 28**

Prompt engineering is the practical skill of communicating with generative AI systems so they produce what you actually need. It is the most immediately employable thing in this course — and the most commonly done badly, because it looks like ordinary conversation and is not.

```{note}
This unit is the one place where the **AI Literacy Program** goes deeper than this book. Modules 5 and 6 of [AI Literacy](https://lndecastro.github.io/AI_Literacy/intro.html) cover prompt anatomy, prompt types, iterative prompting, and context engineering in extended detail. **Read them alongside this unit** — the material here assumes and builds on them rather than repeating them.
```

## Learning Objectives

After completing this unit, you will be able to:

- Decompose a prompt into its functional **components** and explain what each contributes.
- Apply **zero-shot, few-shot, and chain-of-thought** techniques appropriately.
- Design **context** — role, audience, constraints, and format — deliberately rather than accidentally.
- Evaluate prompt quality systematically instead of by impression.

## Part I — Why Prompting Is Not Conversation

A language model does not understand your request. It predicts likely continuations of the text you provide, conditioned on patterns absorbed during training. This has a direct practical consequence:

> **Your prompt does not instruct the model. It positions the model.** You are selecting a region of its learned distribution — steering toward the kind of text that tends to follow prompts like yours.

That is why "write like a senior epidemiologist reviewing a manuscript" outperforms "write well." The first specifies a *region of text*; the second specifies nothing.

### 1.1 Anatomy of an Effective Prompt

| Component | Purpose | Example |
| :--- | :--- | :--- |
| **Role** | Positions expertise and register | "You are a grant reviewer for NSF." |
| **Task** | The specific action required | "Identify the three weakest claims." |
| **Context** | Background the model cannot infer | "This is for an undergraduate audience." |
| **Constraints** | Boundaries on the output | "Under 200 words. No jargon." |
| **Format** | Structure of the response | "Return a numbered list with a one-line rationale each." |
| **Examples** | Demonstrations of what you want | "Here are two well-written entries: ..." |

Most weak prompts contain only **Task**. Most strong prompts contain four or more components.

### 1.2 Core Techniques

- **Zero-shot** — ask directly with no examples. Fine for common, well-defined tasks.
- **Few-shot** — supply two to five worked examples. Dramatically improves format adherence and edge-case handling.
- **Chain-of-thought** — ask the model to reason step by step before answering. Improves multi-step arithmetic and logic.
- **Decomposition** — break a complex request into a sequence of smaller prompts.
- **Self-critique** — ask the model to evaluate and revise its own draft against stated criteria.

```{important}
**Few-shot examples are the highest-leverage technique available to you.** If you want a specific output format, showing two examples of it works better than describing it in three paragraphs. This is the single change that most improves student prompts.
```

## Part II — Context Engineering

Prompting is the message; **context engineering** is everything surrounding it — what the model can see, remember, and reference. As systems gain document upload, tool access, and long memory, context design increasingly matters more than phrasing.

The four questions of context design:

1. **Purpose** — what decision will this output support?
2. **Audience** — who reads it, and what do they already know?
3. **Tone and register** — formal, exploratory, skeptical, instructional?
4. **Boundaries** — what must the model *not* do, assume, or invent?

## ⚙️ Hands-On 1: Comparing Prompts Systematically

You cannot improve what you do not measure. This exercise makes prompt quality visible by testing variations on the same task and scoring the results against fixed criteria.

**Step 1** — copy these four prompts into any AI assistant (Claude, ChatGPT, Gemini, Copilot), one at a time, in separate conversations.

```python
# Run this cell to print four prompt variants for the SAME task.
# Paste each into an AI assistant separately, then score the outputs.

TASK = "explain overfitting to a first-year business student"

prompts = {
    "A — bare task": f"Explain {TASK.split('to')[0].strip()}.",

    "B — role + audience": (
        "You are a professor teaching non-technical undergraduates. "
        f"Explain {TASK.split('to')[0].strip()} to a first-year business student."
    ),

    "C — + constraints + format": (
        "You are a professor teaching non-technical undergraduates.\n"
        f"Task: explain {TASK.split('to')[0].strip()} to a first-year business student.\n"
        "Constraints: under 120 words, no equations, exactly one analogy "
        "drawn from business or everyday life.\n"
        "Format: one short paragraph, then a single bolded takeaway sentence."
    ),

    "D — + few-shot": (
        "You are a professor teaching non-technical undergraduates.\n"
        "Here is the style I want, demonstrated on a different concept:\n\n"
        "CONCEPT: Sampling bias\n"
        "A survey of gym members will tell you a lot about gym members and "
        "almost nothing about the general public. The sample was never "
        "representative, so the conclusions cannot be either.\n"
        "**Takeaway: who you measure determines what you can claim.**\n\n"
        f"Now do the same for: overfitting.\n"
        "Constraints: under 120 words, no equations, one analogy."
    ),
}

for name, p in prompts.items():
    print("=" * 68)
    print(name)
    print("=" * 68)
    print(p, "\n")
```

**Step 2** — score each output on this rubric, 1–5 per criterion:

| Criterion | What you are judging |
| :--- | :--- |
| **Accuracy** | Is the explanation technically correct? |
| **Audience fit** | Would a first-year business student follow it? |
| **Format compliance** | Did it respect the length and structure asked for? |
| **Usefulness** | Would you actually hand this to a student? |

**Step 3** — record which components produced the largest jump. In most trials the biggest single improvement comes between **B and C** (adding constraints and format), with **D** producing the most consistent output across repeated runs.

## ⚙️ Hands-On 2: Prompting a Model Programmatically

You can also call a model from code, which makes systematic comparison much faster. This snippet uses a free local approach — no API key, no account.

First, install the library. In **Colab**, run this in its own cell (the leading `!` is Colab shorthand for a terminal command). Outside Colab, run `pip install transformers` in your terminal instead.

```ipython3
!pip install -q transformers
```

Then run the model — about 500 MB and roughly a minute on first load:

```python
from transformers import pipeline

generator = pipeline("text2text-generation", model="google/flan-t5-base")

prompts = [
    "Summarize: Machine learning models can memorize training data instead "
    "of learning general patterns, which hurts performance on new data.",

    "Answer step by step. A store sells 40 items on Monday and doubles "
    "sales each day. How many items are sold on Thursday?",

    "Classify the sentiment as positive or negative: "
    "The course was demanding but I learned more than I expected.",
]

for p in prompts:
    out = generator(p, max_length=80)[0]["generated_text"]
    print(f"PROMPT: {p[:60]}...")
    print(f"OUTPUT: {out}\n")
```

```{note}
`flan-t5-base` is a **small** model — roughly 250 million parameters, against hundreds of billions in frontier systems. Its answers will often be weak or wrong. That is pedagogically useful: it shows you that prompting technique cannot rescue insufficient capability, and it lets you feel the difference in scale directly.
```

**Try changing it:**

1. Rerun the arithmetic prompt without "Answer step by step." Does chain-of-thought help a model this small?
2. Give the sentiment task two few-shot examples. Does consistency improve?
3. Compare each output to the same prompt in a frontier assistant. Where is the gap largest — reasoning, format, or fluency?

## 💡 Example: Weak Prompt vs. Strong Prompt

**Weak:**
> Write about AI ethics.

**Strong:**
> You are preparing a briefing for a hospital board with no technical background. Write a 250-word summary of the three most significant ethical risks of deploying an AI triage system in an emergency department. For each risk, state the concern in one sentence and one concrete mitigation. Avoid technical jargon. Do not discuss general AI ethics — stay specific to emergency triage.

The second is not longer for the sake of length. Every added clause **removes a class of unwanted outputs.**

## 🧭 Reflection

> If a well-designed prompt reliably produces better output than a poor one, what exactly is the skill being exercised — technical, rhetorical, or editorial?
>
> Prompting resembles delegating to a capable but context-blind assistant. Does getting better at it make you better at delegating to people?

**Connecting to HW7 (Prompt Engineering Exercise, due Oct 28):** submit your four prompt variants, the four outputs, your rubric scores, and a short analysis of which component mattered most and why. **Remember the AI disclosure requirement — see Appendix C.**

## 📘 Further Reading

- Dendritic Institute (2025). _AI Literacy Series — Module 5: Fundamentals of Prompt Engineering._ **(Primary reading for this unit)**
- Dendritic Institute (2025). _AI Literacy Series — Module 6: Context Engineering._ **(Primary reading for this unit)**
- Wei, J. et al. (2022). Chain-of-Thought Prompting Elicits Reasoning in Large Language Models. _arXiv:2201.11903_.
- Mitchell, M. (2020). _AI: A Guide for Thinking Humans_, Ch. 11–13. Penguin Books.
