# Unit 5: Deep Learning and Neural Networks

> **Sessions 13–15** · Sep 30, Oct 05, Oct 07 · **HW5 assigned Oct 05, due Oct 12**

Neural networks are behind nearly every AI capability that has surprised the public in the last decade — image recognition, speech, translation, and the language models you use daily. They are also widely mystified. This unit strips the mystery away: a neural network is a stack of simple arithmetic operations whose numbers are adjusted by a search procedure. That is genuinely all it is. What makes it remarkable is what emerges from doing that at scale.

## Learning Objectives

After completing this unit, you will be able to:

- Describe the **structure** of a neural network: neurons, weights, layers, activation functions.
- Explain **how a network learns** — loss, gradient descent, and backpropagation — in plain language.
- Train networks of varying depth and observe the effect on performance.
- Articulate concrete **limitations** of deep learning and demonstrate at least one failure mode.

## Part I — Structure and Function

### 1.1 The Artificial Neuron

The building block dates to 1943 (McCulloch and Pitts — Unit 1). A single neuron does exactly three things:

1. **Multiply** each input by a weight.
2. **Sum** the results and add a bias.
3. **Squash** the sum through an activation function.

$$\text{output} = f\left(\sum_{i} w_i x_i + b\right)$$

That is the whole operation. The intelligence is not in the neuron — it is in the **values of the weights**, and those are learned, not written.

### 1.2 From Neuron to Network

- **Input layer** → receives the raw features
- **Hidden layers** → transform representations; "deep" simply means *more than one*
- **Output layer** → produces the prediction

```{note}
**Why does depth help?** Each layer learns features built from the previous layer's features. In an image network, early layers detect edges, middle layers detect shapes like eyes and wheels, later layers detect whole objects. Nobody designed that hierarchy — it emerges from training. This is called **representation learning**, and it is the core reason deep learning displaced hand-engineered features.
```

### 1.3 Activation Functions

Without a non-linear activation, stacking layers is pointless — a chain of linear operations collapses into a single linear operation. The non-linearity is what makes depth meaningful.

| Function | Behavior | Where used |
| :--- | :--- | :--- |
| **ReLU** | Zero if negative, unchanged if positive | Default for hidden layers |
| **Sigmoid** | Squashes into (0, 1) | Binary output probabilities |
| **Tanh** | Squashes into (−1, 1) | Older architectures, some RNNs |
| **Softmax** | Turns scores into probabilities summing to 1 | Multi-class output layers |

### 1.4 How Learning Happens

Four steps, repeated thousands of times:

1. **Forward pass** — push an example through, get a prediction.
2. **Loss** — measure how wrong that prediction was.
3. **Backpropagation** — compute how much each weight contributed to the error.
4. **Gradient descent** — nudge every weight slightly in the direction that reduces the error.

> Picture a hiker in fog trying to reach a valley floor. She cannot see the landscape, but she can feel the slope beneath her feet, and steps downhill. Repeat for long enough and she arrives somewhere low. **Gradient descent is that hiker.** The "learning rate" is her step size — too small and she never arrives; too large and she bounds straight over the valley.

## ⚙️ Hands-On 1: A Neuron Learning, Line by Line

Before using a library, build one neuron from scratch. Fifteen lines, no framework.

```python
import numpy as np, matplotlib.pyplot as plt

# Task: learn the logical AND function
X = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
y = np.array([0, 0, 0, 1])

rng = np.random.default_rng(1)
w = rng.normal(0, 0.5, 2)        # weights  -- start random
b = 0.0                           # bias
LR = 0.5                          # learning rate

def sigmoid(z):
    return 1 / (1 + np.exp(-z))

history = []
for epoch in range(400):
    z = X @ w + b                 # 1. FORWARD PASS
    pred = sigmoid(z)
    error = pred - y              # 2. LOSS (how wrong?)
    history.append(np.mean(error ** 2))

    grad_w = X.T @ error / len(y) # 3. BACKPROP (who caused it?)
    grad_b = np.mean(error)

    w -= LR * grad_w              # 4. GRADIENT DESCENT (fix it)
    b -= LR * grad_b

print(f"Learned weights: {np.round(w, 2)}   bias: {b:.2f}\n")
for inputs, p, target in zip(X, sigmoid(X @ w + b), y):
    print(f"  {inputs} -> {p:.3f}   (target {target})")

plt.figure(figsize=(8, 4))
plt.plot(history, lw=2)
plt.xlabel("Epoch"); plt.ylabel("Error")
plt.title("A single neuron learning AND", weight="bold")
plt.tight_layout(); plt.show()
```

Watch the error curve fall. **That descent is learning** — there is nothing else happening.

**Try changing it:**

1. Change `y` to `[0, 1, 1, 1]` (logical OR). It learns this too.
2. Now try `y = [0, 1, 1, 0]` — **XOR**. Run it. The error plateaus and never converges. A single neuron *cannot* learn XOR, because no single straight line separates those points. This exact limitation, published by Minsky and Papert in 1969, helped trigger the first AI winter (Unit 1).
3. Set `LR = 10.0`. Watch the training destabilize. Then set `LR = 0.001` — watch it crawl.

## ⚙️ Hands-On 2: Depth Solves What Width Cannot

XOR was impossible above. Add one hidden layer and it becomes trivial. **This is why depth matters**, demonstrated in the smallest possible case.

```python
import numpy as np, matplotlib.pyplot as plt
from sklearn.neural_network import MLPClassifier
from sklearn.datasets import make_moons
from sklearn.model_selection import train_test_split

# --- First: XOR, now with a hidden layer --------------------------
X_xor = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
y_xor = np.array([0, 1, 1, 0])

net = MLPClassifier(hidden_layer_sizes=(4,), activation="tanh",
                    max_iter=8000, random_state=1).fit(X_xor, y_xor)
print("XOR with a hidden layer:", net.predict(X_xor), " target:", y_xor)

# --- Then: a harder, curved dataset -------------------------------
X, y = make_moons(n_samples=400, noise=0.22, random_state=3)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.3, random_state=3)

architectures = [(1,), (3,), (10,), (30, 30)]
fig, axes = plt.subplots(1, 4, figsize=(18, 4.2))

for ax, arch in zip(axes, architectures):
    clf = MLPClassifier(hidden_layer_sizes=arch, max_iter=4000,
                        random_state=3).fit(X_tr, y_tr)
    # draw the decision boundary
    xx, yy = np.meshgrid(np.linspace(X[:,0].min()-.5, X[:,0].max()+.5, 250),
                         np.linspace(X[:,1].min()-.5, X[:,1].max()+.5, 250))
    Z = clf.predict(np.c_[xx.ravel(), yy.ravel()]).reshape(xx.shape)
    ax.contourf(xx, yy, Z, alpha=0.3, cmap="coolwarm")
    ax.scatter(X_te[:,0], X_te[:,1], c=y_te, cmap="coolwarm",
               edgecolor="k", s=28)
    ax.set_title(f"hidden={arch}\ntest acc {clf.score(X_te, y_te):.1%}")
    ax.set_xticks([]); ax.set_yticks([])

plt.suptitle("More neurons = more flexible boundaries", fontsize=14, weight="bold")
plt.tight_layout(); plt.show()
```

Watch the boundary evolve from a straight line to a smooth curve that follows the data's shape.

**Try changing it:**

1. Set `noise=0.45` and use `hidden_layer_sizes=(200, 200)`. The boundary becomes jagged, contorting to capture noise. **That is Unit 4's overfitting, now in a neural network.** Same disease, bigger model.
2. Compare `(30, 30)` against `(900,)` — roughly similar parameter counts, very different shapes. Does depth or width help more here?
3. Set `max_iter=50`. The network stops before converging. This is **underfitting by impatience**.

## ⚙️ Hands-On 3: Recognizing Handwritten Digits

Now something that feels like real AI — and it still runs in seconds.

```python
import matplotlib.pyplot as plt
from sklearn.datasets import load_digits
from sklearn.neural_network import MLPClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import ConfusionMatrixDisplay, classification_report

digits = load_digits()
X, y = digits.data / 16.0, digits.target      # scale pixels to 0-1
X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.3, random_state=0)

net = MLPClassifier(hidden_layer_sizes=(64, 32), max_iter=600,
                    random_state=0).fit(X_tr, y_tr)

print(f"Test accuracy: {net.score(X_te, y_te):.1%}")
print(classification_report(y_te, net.predict(X_te), digits=3))

# Show some predictions, marking errors in red
fig, axes = plt.subplots(2, 8, figsize=(15, 4.2))
preds = net.predict(X_te)
for i, ax in enumerate(axes.ravel()):
    ax.imshow(X_te[i].reshape(8, 8), cmap="gray_r")
    ok = preds[i] == y_te[i]
    ax.set_title(f"{preds[i]}", color="green" if ok else "red",
                 fontsize=13, weight="bold")
    ax.axis("off")
plt.suptitle("Predictions (red = wrong)", fontsize=13, weight="bold")
plt.tight_layout(); plt.show()

ConfusionMatrixDisplay.from_estimator(net, X_te, y_te, cmap="Blues")
plt.title("Which digits get confused?"); plt.show()
```

Look at the confusion matrix. The errors are not random — 4 and 9 get mixed, 3 and 8 get mixed. **The model's confusions are structurally similar to the ones humans make**, because it is picking up on genuinely ambiguous shapes.

## Part II — Applications and Limitations

Deep learning's successes are real: medical imaging, protein folding, speech recognition, machine translation, generative models. But every one of these is a **narrow** success, and understanding the boundaries is the more valuable skill.

### 2.1 Known Limitations

| Limitation | What it means |
| :--- | :--- |
| **Data hunger** | Needs vast labeled examples; humans generalize from a handful |
| **Brittleness** | Small, meaningless input changes can flip the output entirely |
| **Opacity** | Millions of weights, no readable explanation |
| **No causal model** | Learns correlation, not cause |
| **Distribution shift** | Performance collapses when the world changes |
| **Confident errors** | Wrong answers arrive with the same confidence as right ones |

### ⚙️ Hands-On 4: Making the Model Fail

This is the most important experiment in the unit. Take the digit classifier that just scored ~97% and shift every image by two pixels — a change any human would not even notice.

```python
import numpy as np, matplotlib.pyplot as plt

# Shift each 8x8 image two pixels to the right
X_te_img = X_te.reshape(-1, 8, 8)
X_shift = np.roll(X_te_img, shift=2, axis=2).reshape(-1, 64)

acc_orig  = net.score(X_te, y_te)
acc_shift = net.score(X_shift, y_te)

print(f"Accuracy on original images: {acc_orig:.1%}")
print(f"Accuracy after 2-pixel shift: {acc_shift:.1%}")
print(f"Performance lost: {acc_orig - acc_shift:.1%}")

fig, axes = plt.subplots(2, 6, figsize=(12, 4.5))
for i in range(6):
    axes[0, i].imshow(X_te_img[i], cmap="gray_r"); axes[0, i].axis("off")
    axes[0, i].set_title(f"pred {net.predict(X_te[i:i+1])[0]}", color="green")
    axes[1, i].imshow(X_shift[i].reshape(8, 8), cmap="gray_r"); axes[1, i].axis("off")
    axes[1, i].set_title(f"pred {net.predict(X_shift[i:i+1])[0]}", color="red")
axes[0, 0].set_ylabel("original"); axes[1, 0].set_ylabel("shifted")
plt.suptitle("Same digits, two pixels over — and the model collapses",
             fontsize=13, weight="bold")
plt.tight_layout(); plt.show()
```

Accuracy falls from roughly **98% to about 17%** — worse than it would do by memorizing a single digit and guessing that every time. **You can still read every digit perfectly.**

Worse still, run this:

```python
conf_orig  = net.predict_proba(X_te).max(axis=1).mean()
conf_shift = net.predict_proba(X_shift).max(axis=1).mean()
print(f"Average confidence, original images: {conf_orig:.1%}")
print(f"Average confidence, shifted images:  {conf_shift:.1%}")
```

The model is roughly **82% confident** in answers that are wrong 83% of the time. It has no mechanism for noticing that it left familiar territory.

```{warning}
This is the most important thing to take from Unit 5. The network never learned "what a 3 looks like." It learned **which specific pixel positions tend to be dark in training images labeled 3**. Those are not the same thing, and no amount of accuracy on the test set reveals the difference.

When you read that a model achieves "superhuman performance," ask immediately: *superhuman on what distribution?* Move even slightly outside it and the performance is not merely reduced — it can vanish.
```

**Try changing it:**

1. Reduce the shift to 1 pixel (`shift=1`). Accuracy still falls to roughly 49% — **half the model's skill destroyed by a one-pixel translation.** Try 3 pixels to see it bottom out.
2. Retrain the model with shifted images *added* to the training set. Accuracy recovers — this is **data augmentation**, standard practice precisely because of this fragility.
3. A system that is confidently wrong is more dangerous than one that admits uncertainty. Where would that property matter most: medical triage, loan approval, or content recommendation? This connects directly to Unit 11.

## 🧭 Reflection

> A network with 97% accuracy fails on images a child reads effortlessly. What does "understanding" mean here, and is the word appropriate at all?
>
> Deep learning's power comes from learning its own representations, without human design. What does that imply for our ability to audit, explain, or contest an AI decision?

**Connecting to HW5 (Neural Network Reflection, due Oct 12):** the failure experiment in Hands-On 4 is the strongest possible material for this assignment. Report your actual numbers.

## 📘 Further Reading

- Goodfellow, I., Bengio, Y., & Courville, A. (2016). _Deep Learning._ MIT Press.
- Mitchell, M. (2020). _Artificial Intelligence: A Guide for Thinking Humans_, Ch. 4–6. Penguin Books.
- Russell, S., & Norvig, P. (2022). _AI: A Modern Approach_, 4th Ed., Ch. 21. Pearson.
- Marcus, G. (2018). Deep Learning: A Critical Appraisal. _arXiv:1801.00631_.
