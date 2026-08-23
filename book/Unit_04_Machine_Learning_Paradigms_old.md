# Unit 4: Machine Learning Paradigms

> **Sessions 9–12** · Sep 16, 21, 23, 28 · **HW4 assigned Sep 23, due Sep 30**

Machine learning is not one technique. It is a family of approaches distinguished by **what kind of information the machine receives** and **what it is asked to do with it**. This unit is where the vocabulary of Units 1–3 becomes something you can manipulate with your own hands.

## Learning Objectives

After completing this unit, you will be able to:

- Distinguish the four main **learning paradigms** — supervised, unsupervised, semi-supervised, and reinforcement — and identify which fits a given problem.
- Explain the difference between **classification** and **clustering** by applying both to the same data.
- Describe the **exploration/exploitation trade-off** in reinforcement learning.
- Recognize **overfitting** by observing the divergence of training and test error.

## Part I — The Four Paradigms

### 1.1 Supervised Learning

The most common and best-understood paradigm.

- The model learns from **labeled data** — examples where the correct output is known.
- Objective: find a function mapping inputs (features) to outputs (labels).
- Examples: predicting house prices (**regression**); classifying email as spam (**classification**).

**Common algorithms:** Linear/Logistic Regression, Decision Trees, Random Forests, Support Vector Machines, Neural Networks.

> Supervised learning mirrors **learning by example** — the machine imitates patterns it has been shown.

### 1.2 Unsupervised Learning

- Works with **unlabeled data** — the algorithm must discover structure on its own.
- Objective: reveal hidden relationships or group similar points.
- Examples: customer segmentation, topic discovery, dimensionality reduction.

**Common techniques:** Clustering (K-Means, DBSCAN, hierarchical), Association Rules, Principal Component Analysis.

> Unsupervised learning is about **exploration** — letting the data reveal its own organization.

### 1.3 Semi-Supervised Learning

- Combines a **small amount of labeled data** with a large amount of unlabeled data.
- Useful whenever labeling is expensive: medical images, legal documents, scientific data.

> Semi-supervised learning reflects a realistic compromise — **humans label some data, the machine infers the rest.**

### 1.4 Reinforcement Learning

- Inspired by behavioral psychology: learning through **trial, error, and feedback**.
- An **agent** acts in an **environment**, receiving **rewards** or **penalties**, and learns a **policy** maximizing long-term reward.
- Applications: game playing (AlphaGo), robotics, autonomous driving, resource allocation.

**Core components:** Agent → Environment → State → Action → Reward.

> Reinforcement learning mirrors how humans and animals actually learn: **by doing, failing, and improving.**

### Summary

| Paradigm | Data provided | Question answered |
| :--- | :--- | :--- |
| **Supervised** | Features **+ labels** | "What is the correct answer for this input?" |
| **Unsupervised** | Features only | "What structure exists in this data?" |
| **Semi-supervised** | Few labels, many unlabeled | "Can I stretch a small labeled set?" |
| **Reinforcement** | Rewards from an environment | "Which sequence of actions pays off?" |

## ⚙️ Hands-On 1: Same Data, Two Paradigms

The clearest way to feel the supervised/unsupervised distinction is to give both the **identical dataset** and watch them answer different questions.

```python
import numpy as np, matplotlib.pyplot as plt
from sklearn.datasets import make_blobs
from sklearn.neighbors import KNeighborsClassifier
from sklearn.cluster import KMeans
from sklearn.model_selection import train_test_split

# One dataset, three natural groups
X, y_true = make_blobs(n_samples=300, centers=3, cluster_std=1.3,
                       random_state=7)

# --- SUPERVISED: we GIVE it the labels ----------------------------
X_tr, X_te, y_tr, y_te = train_test_split(X, y_true, test_size=0.3,
                                          random_state=7)
knn = KNeighborsClassifier(n_neighbors=5).fit(X_tr, y_tr)
print(f"Supervised test accuracy: {knn.score(X_te, y_te):.1%}")

# --- UNSUPERVISED: we HIDE the labels -----------------------------
kmeans = KMeans(n_clusters=3, n_init=10, random_state=7).fit(X)
y_found = kmeans.labels_

fig, axes = plt.subplots(1, 3, figsize=(16, 4.5))
axes[0].scatter(X[:, 0], X[:, 1], c="gray", s=25)
axes[0].set_title("What the machine sees\n(unsupervised: no labels)")

axes[1].scatter(X[:, 0], X[:, 1], c=y_true, cmap="viridis", s=25)
axes[1].set_title("True labels\n(supervised: these are provided)")

axes[2].scatter(X[:, 0], X[:, 1], c=y_found, cmap="viridis", s=25)
axes[2].scatter(*kmeans.cluster_centers_.T, c="red", marker="X", s=250,
                edgecolor="black", linewidth=1.5)
axes[2].set_title("Groups K-Means DISCOVERED\n(red X = cluster centers)")

for ax in axes:
    ax.set_xticks([]); ax.set_yticks([])
plt.tight_layout(); plt.show()
```

**Look carefully at panels 2 and 3.** The groupings are nearly identical — but the *colors don't match*. K-Means found the structure without ever being told the groups existed, yet it has no idea what to *call* them. It found "these three piles"; it did not find "setosa, versicolor, virginica."

```{important}
This is the essential difference. **Supervised learning learns a name. Unsupervised learning learns a shape.** Clustering can tell you that your customers fall into four groups; only a human can tell you what those groups mean.
```

**Try changing it:**

1. Set `n_clusters=5` while the data truly has 3 groups. K-Means will happily produce five clusters. **It never tells you that you asked the wrong question** — a critical limitation.
2. Raise `cluster_std` to 3.0 so the groups overlap. Which paradigm degrades faster?
3. Reduce the supervised training set to 10 points (`test_size=0.97`). How much labeled data do you actually need?

## ⚙️ Hands-On 2: Reinforcement Learning — The Explore/Exploit Problem

Reinforcement learning is easiest to grasp in its simplest form: a **multi-armed bandit**. Imagine three slot machines with unknown payout rates. Every pull is a choice between **exploiting** the machine that has looked best so far and **exploring** one you know less about.

```python
import numpy as np, matplotlib.pyplot as plt

rng = np.random.default_rng(42)
TRUE_RATES = [0.25, 0.55, 0.40]      # hidden from the agent!
N_PULLS = 1000

def run_bandit(epsilon):
    """epsilon = probability of EXPLORING instead of exploiting."""
    counts = np.zeros(3)
    est = np.zeros(3)                 # agent's current beliefs
    rewards = []
    for _ in range(N_PULLS):
        if rng.random() < epsilon:
            arm = rng.integers(3)                 # EXPLORE
        else:
            arm = int(np.argmax(est))             # EXPLOIT
        reward = 1 if rng.random() < TRUE_RATES[arm] else 0
        counts[arm] += 1
        est[arm] += (reward - est[arm]) / counts[arm]   # update belief
        rewards.append(reward)
    return np.cumsum(rewards) / np.arange(1, N_PULLS + 1), est, counts

plt.figure(figsize=(11, 5))
for eps, label in [(0.0, "ε=0.0  pure exploitation"),
                   (0.1, "ε=0.1  balanced"),
                   (0.5, "ε=0.5  mostly exploring")]:
    avg, est, counts = run_bandit(eps)
    plt.plot(avg, label=label, linewidth=2)
    print(f"ε={eps}: final beliefs {np.round(est,2)}  pulls {counts.astype(int)}")

plt.axhline(max(TRUE_RATES), color="black", ls="--",
            label=f"best possible ({max(TRUE_RATES)})")
plt.xlabel("Number of pulls"); plt.ylabel("Average reward so far")
plt.title("Exploration vs. Exploitation", fontsize=13, weight="bold")
plt.legend(); plt.tight_layout(); plt.show()
```

Run it several times. The pattern that emerges:

- **ε = 0** (never explore) sometimes locks onto a mediocre machine forever, because it stopped gathering evidence too early.
- **ε = 0.5** (explore constantly) learns the true rates accurately but wastes half its pulls on machines it knows are bad.
- **ε = 0.1** usually wins.

> This trade-off is not a quirk of slot machines. It is the structure of **every decision made under uncertainty** — choosing a restaurant, a research direction, a career. Should you take the known-good option, or gather information about alternatives? Reinforcement learning gives that intuition a precise mathematical form.

**Try changing it:**

1. Make two machines nearly identical (`[0.50, 0.52, 0.30]`). How much harder does the problem become?
2. Add a *decaying* epsilon — explore a lot early, then settle down. Does it beat a fixed ε?
3. Change `N_PULLS` to 50. With few attempts, which strategy wins? What does that say about learning in short-lived situations?

## ⚙️ Hands-On 3: Watching a Model Overfit

This is the most important activity in the unit. **Overfitting** is when a model memorizes its training data instead of learning the underlying pattern — and here you will watch it happen.

```python
import numpy as np, matplotlib.pyplot as plt
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.pipeline import make_pipeline
from sklearn.metrics import mean_squared_error

rng = np.random.default_rng(0)

# True underlying pattern: a gentle wave. Reality adds noise.
def true_pattern(x):
    return np.sin(1.5 * np.pi * x)

X_train = np.sort(rng.uniform(0, 1, 25))
y_train = true_pattern(X_train) + rng.normal(0, 0.25, 25)
X_test  = np.sort(rng.uniform(0, 1, 100))
y_test  = true_pattern(X_test) + rng.normal(0, 0.25, 100)

X_plot = np.linspace(0, 1, 300)

# ---- CHANGE THIS NUMBER: try 1, 3, 6, 10, 15, 20 -----------------
DEGREE = 15
# ------------------------------------------------------------------

model = make_pipeline(PolynomialFeatures(DEGREE), LinearRegression())
model.fit(X_train.reshape(-1, 1), y_train)

tr_err = mean_squared_error(y_train, model.predict(X_train.reshape(-1, 1)))
te_err = mean_squared_error(y_test,  model.predict(X_test.reshape(-1, 1)))

plt.figure(figsize=(9, 5.5))
plt.scatter(X_train, y_train, s=55, edgecolor="black", zorder=5,
            label="training data (25 points)")
plt.plot(X_plot, true_pattern(X_plot), "g--", lw=2, label="true pattern")
plt.plot(X_plot, model.predict(X_plot.reshape(-1, 1)), "r-", lw=2,
         label=f"model (degree {DEGREE})")
plt.ylim(-2, 2)
plt.title(f"Degree {DEGREE} | train error {tr_err:.3f} | "
          f"TEST error {te_err:.3f}", fontsize=12, weight="bold")
plt.legend(); plt.tight_layout(); plt.show()
```

Run this **six times**, changing `DEGREE` each time to 1, 3, 6, 10, 15, 20. Record the two error numbers each run.

You will see three regimes:

| Degree | Behavior | Name |
| :--- | :--- | :--- |
| 1 | Too rigid — a straight line through a curve. Both errors high. | **Underfitting** |
| 3–6 | Follows the true pattern. Both errors low. | **Good fit** |
| 15–20 | Passes through nearly every training point. Train error tiny, **test error explodes**. | **Overfitting** |

Now watch the whole curve at once:

```python
degrees = range(1, 21)
train_errs, test_errs = [], []

for d in degrees:
    m = make_pipeline(PolynomialFeatures(d), LinearRegression())
    m.fit(X_train.reshape(-1, 1), y_train)
    train_errs.append(mean_squared_error(y_train, m.predict(X_train.reshape(-1, 1))))
    test_errs.append(mean_squared_error(y_test, m.predict(X_test.reshape(-1, 1))))

plt.figure(figsize=(10, 5.5))
plt.plot(degrees, train_errs, "o-", lw=2, label="Training error")
plt.plot(degrees, test_errs, "s-", lw=2, label="Test error")
plt.yscale("log")
best = int(np.argmin(test_errs)) + 1
plt.axvline(best, color="green", ls="--", label=f"sweet spot (degree {best})")
plt.xlabel("Model complexity (polynomial degree)")
plt.ylabel("Error (log scale)")
plt.title("The Central Trade-off in Machine Learning", fontsize=13, weight="bold")
plt.xticks(list(degrees)); plt.legend(); plt.tight_layout(); plt.show()
```

```{important}
**This graph is the single most important picture in machine learning.** Training error falls forever as complexity grows — you can always memorize harder. Test error falls, bottoms out, then rises. The gap between the two curves *is* overfitting. Every technique you will hear about later — regularization, dropout, early stopping, cross-validation — exists to manage this one graph.
```

**Try changing it:**

1. Increase the training set from 25 points to 200. Watch the overfitting region shrink dramatically. **More data is the most reliable cure.**
2. Set the noise to 0.0. Does high complexity still hurt? Why or why not?
3. If you only ever looked at training error, which degree would you pick? What would that cost you?

## 🧭 Reflection

> A model that memorizes gets a perfect score on material it has already seen and fails on anything new.
> Does that remind you of any approach to studying?
>
> K-Means found three groups without being told they existed — but could not name them. What does that suggest about which parts of interpretation remain irreducibly human?

**Connecting to HW4 (ML Paradigms — Compare & Contrast, due Sep 30):** use the outputs you generated here as evidence. A comparison grounded in something you actually ran is far stronger than one paraphrased from a textbook.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 19–22. Pearson.
- Mitchell, T. M. (1997). _Machine Learning._ McGraw-Hill.
- Sutton, R. S., & Barto, A. G. (2018). _Reinforcement Learning: An Introduction_, 2nd Ed. MIT Press.
- Dendritic Institute (2025). _AI Literacy Series — Module 3: How Machines Learn._
