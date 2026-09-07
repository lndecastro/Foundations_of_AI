# Unit 5: Deep Learning and Neural Networks

> **Sessions 8–9** · Sep 14, 16 · **HW5 assigned Sep 16, due Sep 23**

Every tool you will use in Parts III, IV, and V of this course — the chat assistants, the presentation generators, the coding tools — is a neural network underneath. This unit is the last time we open the box before we start using what is inside it.

You do not need the mathematics. You do need three things: **what a neuron computes**, **why depth changed everything**, and **why these systems fail the specific way they do**. That last one is what separates a professional user of AI tools from a credulous one.

## Learning Objectives

After completing this unit, you will be able to:

- Describe the components of an **artificial neuron** and what training actually adjusts.
- Explain why a single-layer network cannot solve XOR, and what that failure did to the field.
- Build and train a **multilayer network**, and relate its capacity to its parameter count.
- Explain, in mechanical terms, **why language models produce fluent falsehoods**.

## Part I — From Neuron to Network

### 1.1 What a Single Neuron Does

An artificial neuron performs three operations, in order:

1. **Weight** each input by a number that expresses how much that input matters.
2. **Sum** the weighted inputs and add a **bias** term.
3. **Activate** — pass the sum through a nonlinear function that decides the output.

That is the entire computation. The intelligence, such as it is, lives in the weights, and the weights are **learned** — adjusted repeatedly so that the network's outputs move closer to the desired ones. Training a neural network is nothing more than searching for a good set of numbers.

```{note}
The **activation function** is the step that matters most and gets explained least. Without it, stacking layers is pointless: a chain of linear operations collapses algebraically into a single linear operation, so a hundred-layer network would have exactly the expressive power of one layer. The nonlinearity is what makes depth mean something.
```

### 1.2 What Training Adjusts

- **Forward pass** — data flows in, a prediction comes out.
- **Loss** — a single number measuring how wrong the prediction was.
- **Backpropagation** — the loss is traced backwards to determine how much each weight contributed to the error.
- **Update** — every weight moves a small step in the direction that reduces the loss.

Repeat several million times. There is no moment of insight anywhere in this process.

### 1.3 The Failure That Caused a Winter

In Unit 1 you discussed the AI winters. Here is the technical event behind the first one.

The **perceptron** (Rosenblatt, 1958) was a single-layer network, and it generated enormous excitement. In 1969, Minsky and Papert proved it could not learn the XOR function — a problem so small it fits in four rows. Funding collapsed. The field lost roughly fifteen years.

The solution turned out to be adding one hidden layer. The mathematics for training such networks efficiently (backpropagation) took until the 1980s to become widely known. You are about to reproduce both halves of this history in about twenty lines.

## ⚙️ Hands-On: The Problem That Broke the Perceptron

XOR outputs 1 when its two inputs differ and 0 when they match. Four examples, two features. Watch what happens.

```python
import numpy as np
from sklearn.linear_model import Perceptron
from sklearn.neural_network import MLPClassifier
from sklearn.metrics import accuracy_score

X = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
y = np.array([0, 1, 1, 0])                    # XOR: 1 when inputs differ

single = Perceptron(max_iter=1000, random_state=0).fit(X, y)
multi   = MLPClassifier(hidden_layer_sizes=(4,), activation="tanh",
                        max_iter=5000, random_state=1).fit(X, y)

print("input   target   perceptron   MLP(4 hidden)")
for xi, yi, a, b in zip(X, y, single.predict(X), multi.predict(X)):
    print(f"{xi}     {yi}          {a}            {b}")

print(f"\nPerceptron accuracy:      {accuracy_score(y, single.predict(X)):.0%}")
print(f"MLP (1 hidden layer):     {accuracy_score(y, multi.predict(X)):.0%}")
```

The perceptron scores **50%** — it predicts a single class for all four inputs and cannot do better, no matter how long it trains. Adding four hidden neurons takes it to **100%**.

The reason is geometric. A single-layer network can only separate points with one straight line, and no straight line puts `[0,1]` and `[1,0]` on one side with `[0,0]` and `[1,1]` on the other. A hidden layer lets the network bend the space first, and in the bent space a single line suffices.

```{warning}
Fifteen years of funding disappeared over a limitation that one extra layer removes. It is worth asking, whenever you read a confident claim that AI "cannot" do something, whether the claim is about a fundamental limit or about the specific architecture someone happened to test.
```

**Try changing it:**

1. Set `hidden_layer_sizes=(1,)`. Does one hidden neuron suffice? Find the smallest hidden layer that reliably solves XOR.
2. Change `y` to the AND function (`[0,0,0,1]`). Does the perceptron manage this one? Why is AND easier than XOR?
3. Set `activation="identity"` on the MLP. Accuracy collapses back to 50% even with hidden neurons. Explain this using §1.1.

## ⚙️ Hands-On: Capacity, Depth, and What Scale Buys

Now something harder: recognizing handwritten digits from 8×8 images. We vary the size of the network and watch what capacity buys.

```python
import warnings; warnings.filterwarnings("ignore")
import matplotlib.pyplot as plt
from sklearn.datasets import load_digits
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.neural_network import MLPClassifier
from sklearn.metrics import accuracy_score, confusion_matrix

digits = load_digits()
X = StandardScaler().fit_transform(digits.data)
X_tr, X_te, y_tr, y_te = train_test_split(
    X, digits.target, test_size=0.3, random_state=42, stratify=digits.target
)

print(f"{'architecture':>14}  {'weights':>8}  {'train':>7}  {'test':>7}")
best = None
for hidden in [(4,), (16,), (32,), (128, 64)]:
    net = MLPClassifier(hidden_layer_sizes=hidden, max_iter=600,
                        random_state=1).fit(X_tr, y_tr)
    n_w = sum(c.size for c in net.coefs_)
    tr = accuracy_score(y_tr, net.predict(X_tr))
    te = accuracy_score(y_te, net.predict(X_te))
    print(f"{str(hidden):>14}  {n_w:8}  {tr:7.1%}  {te:7.1%}")
    best = net

fig, ax = plt.subplots(1, 2, figsize=(13, 5))
ax[0].imshow(confusion_matrix(y_te, best.predict(X_te)), cmap="Greens")
ax[0].set_title("Confusion matrix — where digits get mixed up")
ax[0].set_xlabel("Predicted"); ax[0].set_ylabel("Actual")

wrong = [i for i, (t, p) in enumerate(zip(y_te, best.predict(X_te))) if t != p]
if wrong:
    idx = wrong[0]
    ax[1].imshow(X_te[idx].reshape(8, 8), cmap="gray")
    ax[1].set_title(f"A mistake: actual {y_te[idx]}, "
                    f"predicted {best.predict(X_te)[idx]}")
    ax[1].axis("off")
plt.tight_layout(); plt.show()
```

Four hidden neurons reach about **87%**. Thirty-two reach about **97%**. Going all the way to 128+64 — roughly seven times more weights — buys well under one additional point.

That shape is the story of modern AI in miniature: **large early gains from capacity, then sharply diminishing returns.** The frontier language models are on the far right of this curve, which is why each new generation costs enormously more to train and feels only somewhat better to use.

**Try changing it:**

1. Look at the misclassified digit the code displays. Would you have read it correctly? Are the model's errors the errors a human would make?
2. Set `max_iter=20`. Accuracy drops sharply. Distinguish a model that lacks capacity from one that simply has not finished training.
3. Add `hidden_layer_sizes=(512, 256, 128)`. Does test accuracy improve enough to justify the cost? You have just performed the calculation every ML team argues about.

## Part II — Why These Systems Hallucinate

This section is the bridge to the rest of the course, and it is the most practically useful thing in the unit.

A large language model is a neural network trained on one objective: **given a sequence of text, predict what comes next**. Not "state the truth." Not "answer correctly." Predict the next token, given everything before it.

Everything that follows is a consequence of that objective.

- **Fluency is guaranteed.** Producing text that reads naturally is precisely what was optimized. The output will be well-formed whether or not it is correct.
- **Truth is incidental.** True statements appear frequently in training data, so they are often the likely continuation. That is a correlation, not a mechanism. Nothing checks.
- **Confidence is not calibrated.** The model has no representation of "I am unsure." It produces the most likely next token either way. A fabricated citation and a real one are generated by identical machinery.
- **Plausible errors are the dangerous ones.** Fabrications tend to be *typical* — a paper title that sounds like a real paper, an API method that ought to exist. These are the hardest errors for a reader to catch, and they are hard precisely *because* the system is good at its job.

```{warning}
"Hallucination" is a misleading word. It suggests a malfunction. Fluent falsehood is the system operating exactly as designed — the design simply never included a truth-checking step. Expecting one is a category error, and building a workflow that assumes one is a professional risk.

Every AI tool you use from Unit 7 onward has this property. The verification habits you build in Part III are not optional politeness. They are the compensating control for a known and permanent limitation.
```

## 💡 Example: Reading a Model Card

When a team ships a model, they publish its architecture, parameter count, training data, and evaluation results. You now have enough vocabulary to read one critically. Three questions worth asking of any such document:

1. **What was the test set, and could it overlap the training data?** Unit 3's golden rule applies at every scale. Contamination inflates every headline number.
2. **Is the reported gain worth the parameter increase?** You measured the shape of that curve yourself.
3. **What failure modes are reported, and which are conspicuously absent?** A model card with no error analysis is marketing.

## 🧭 Reflection

> You trained a network to 97% on handwritten digits. It has no concept of a digit, of writing, or of counting. It has a matrix of numbers that transforms pixel patterns into ten scores.
>
> When a language model produces a paragraph that seems to understand your question, what is the honest reason to believe that anything different is happening?

**Connecting to HW5 (Neural Network Reflection):** find a real example of an AI system producing a confident falsehood — from your own use, from the news, or from deliberately provoking one. Explain the failure **mechanically**, using this unit's vocabulary. Describing *that* it was wrong is not the assignment; explaining *why the architecture makes that error likely* is.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 21. Pearson.
- Mitchell, M. (2020). _Artificial Intelligence: A Guide for Thinking Humans_, Part II, Ch. 4–6. Penguin Books.
- Géron, A. (2022). _Hands-On Machine Learning with Scikit-Learn, Keras, and TensorFlow_, 3rd Ed., Ch. 10–11. O'Reilly.
- Minsky, M., & Papert, S. (1969). _Perceptrons_. MIT Press.
- Dendritic Institute (2025). _AI Literacy Series — Module 1: Understanding AI Without the Fear._
