# Unit 2: Goals and Foundations of AI

> **Sessions 5–6** · Aug 31, Sep 02 · **HW2 assigned Sep 02, due Sep 09**

Before you can evaluate an AI system, you need to know **what it is trying to do**. This unit builds the vocabulary you will use for the rest of the semester — and the vocabulary matters more than it might seem. Most public confusion about AI comes from mixing up categories: calling a chatbot "intelligent," calling a classifier "biased," calling a prediction a "decision." Precision here pays off everywhere later.

## Learning Objectives

After completing this unit, you will be able to:

- Distinguish the **four goals** AI has historically pursued and explain which ones dominate today.
- Define the **rational agent** framework and use it to describe any AI system.
- Classify a real-world problem into the correct **task type** (classification, regression, clustering, generation, and others).
- Use core terminology — model, feature, label, training, inference — correctly and precisely.

## Part I — What Is AI Trying to Do?

### 1.1 Four Historical Goals

Russell and Norvig organize the field's ambitions along two axes: whether the system should **think** or **act**, and whether the standard is **human performance** or **rationality**.

| | **Humanly** | **Rationally** |
| :--- | :--- | :--- |
| **Thinking** | Cognitive modeling — replicate human reasoning | Laws of thought — reason correctly by logic |
| **Acting** | Turing Test — behave indistinguishably from a person | **Rational agents** — do the right thing given the goal |

Modern AI overwhelmingly pursues the bottom-right quadrant: **rational action**. A spam filter is not trying to think like a person about email. It is trying to make the correct sorting decision as often as possible.

> This is why "does the machine really understand?" is often the wrong question for practical purposes — and simultaneously the right question for ethical ones. Hold both.

### 1.2 The Agent Framework

Nearly any AI system can be described with four elements:

- **Environment** → the world the system operates in
- **Sensors / Percepts** → what it can observe
- **Actions / Actuators** → what it can do
- **Performance measure** → how we score whether it did well

```{note}
The **performance measure** is where most real-world AI failures originate. A system optimizes exactly what you measure — not what you meant. A recommendation engine measured on "watch time" will learn to maximize watch time, even if that means promoting outrage. Unit 10 and Unit 11 return to this repeatedly.
```

### 1.3 Types of Environments

| Property | Easier | Harder |
| :--- | :--- | :--- |
| **Observability** | Fully observable (chess) | Partially observable (driving in fog) |
| **Determinism** | Deterministic | Stochastic |
| **Agents** | Single agent | Multi-agent, possibly adversarial |
| **Time** | Episodic (each case independent) | Sequential (actions have consequences) |
| **Change** | Static | Dynamic |

Chess is fully observable, deterministic, and static — which is why it fell to computers first. Driving is none of those things, which is why it remains hard.

## Part II — Core Terminology

These six terms will appear in every remaining unit. Learn them now.

| Term | Meaning | Everyday analogy |
| :--- | :--- | :--- |
| **Feature** | An input variable the system observes | The symptoms a doctor notes |
| **Label** | The correct answer for a given example | The confirmed diagnosis |
| **Model** | The learned mapping from features to output | The doctor's accumulated judgment |
| **Training** | The process of fitting the model to data | Years of medical residency |
| **Inference** | Using the trained model on new input | Seeing a new patient today |
| **Parameters** | The internal numbers adjusted during training | What actually changed in the doctor's head |

> A **model** is not a program someone wrote. It is a set of numbers that were *discovered* by a training process. Nobody chose them, and often nobody can fully explain them. That distinction drives the interpretability problems of Unit 10.

### 2.1 Task Types

Most AI problems reduce to a handful of shapes:

- **Classification** → predict a category. *Is this transaction fraudulent?*
- **Regression** → predict a number. *What will this house sell for?*
- **Clustering** → group similar items with no labels given. *What customer segments exist?*
- **Ranking** → order items by relevance. *Which results go on page one?*
- **Generation** → produce new content. *Write a summary of this report.*
- **Sequential decision-making** → choose actions over time. *Route this delivery fleet.*

Correctly identifying the task type is the first step in every project — including your capstone.

## ⚙️ Hands-On: Building a Rational Agent

Here is a complete rational agent in about twenty lines. It lives in a small grid world, cannot see the whole map, and must find its way to a goal. Watch how the four elements — environment, percepts, actions, performance measure — appear explicitly in the code.

```python
from collections import deque

# --- The ENVIRONMENT: 0 = open, 1 = wall ---------------------------
grid = [
    [0, 0, 0, 1, 0],
    [1, 1, 0, 1, 0],
    [0, 0, 0, 0, 0],
    [0, 1, 1, 1, 0],
    [0, 0, 0, 1, 0],
]
start, goal = (0, 0), (4, 4)

def neighbors(cell):
    """The agent's PERCEPTS: which adjacent squares are open?"""
    r, c = cell
    for dr, dc in [(-1, 0), (1, 0), (0, -1), (0, 1)]:   # its ACTIONS
        nr, nc = r + dr, c + dc
        if 0 <= nr < len(grid) and 0 <= nc < len(grid[0]) and grid[nr][nc] == 0:
            yield (nr, nc)

def find_path(start, goal):
    """Breadth-first search: guarantees the SHORTEST path."""
    queue = deque([[start]])
    visited = {start}
    while queue:
        path = queue.popleft()
        if path[-1] == goal:
            return path
        for nxt in neighbors(path[-1]):
            if nxt not in visited:
                visited.add(nxt)
                queue.append(path + [nxt])
    return None

path = find_path(start, goal)

# --- The PERFORMANCE MEASURE: number of steps taken ----------------
print(f"Path found in {len(path) - 1} steps:")
for r in range(len(grid)):
    row = ""
    for c in range(len(grid[0])):
        if (r, c) == start:   row += " S "
        elif (r, c) == goal:  row += " G "
        elif (r, c) in path:  row += " . "
        elif grid[r][c] == 1: row += " # "
        else:                 row += "   "
    print(row)
```

**Try changing it:**

1. Add a wall that blocks the path entirely. What does the agent do? Is failing gracefully a form of rational behavior?
2. Change the performance measure. What if diagonal moves were allowed but cost twice as much? Would the shortest path still be the *best* path?
3. This agent has **no learning whatsoever** — it searches fresh every time. Is it still "intelligent"? Defend your answer using the four goals from Part I.

```{important}
This agent is **entirely symbolic** — the rules of movement were written by a human. It is a direct descendant of 1970s AI, and it works perfectly here because the environment is small, fully observable, deterministic, and static. Change any one of those and it breaks. That is the boundary line where machine learning becomes necessary.
```

## 💡 Example: Naming the Task Type

For each scenario below, decide the task type *before* reading the answer.

| Scenario | Task type |
| :--- | :--- |
| Flagging tumors in an X-ray | Classification |
| Estimating tomorrow's electricity demand | Regression |
| Grouping students by study habits, no categories given | Clustering |
| Drafting a first-pass contract | Generation |
| Deciding which ad to show, learning from clicks over time | Sequential decision-making |
| Sorting job applicants by fit | Ranking (*and* an ethics problem — see Unit 10) |

That last row is deliberate. **Framing a human decision as a ranking task is itself a consequential choice**, not a neutral technical one.

## 🧭 Reflection

> Pick an AI system you used this week. Write out its environment, percepts, actions, and performance measure.
> Now ask: is the performance measure it optimizes the same as the outcome *you* wanted? Where do they diverge?

**Connecting to HW2 (Vocabulary & Concept Review, due Sep 09):** the six terms in Part II and the six task types in Part 2.1 form the core of this assignment. Practice using them on systems you encounter, not just on the definitions.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 2. Pearson.
- Mitchell, M. (2020). _Artificial Intelligence: A Guide for Thinking Humans_, Ch. 1–3. Penguin Books.
- Dendritic Institute (2025). _AI Literacy Series — Module 2: AI and Its Many Branches._
