# Unit 3: How AI Works

> **Sessions 7–8** · Sep 09, Sep 14 · **HW3 assigned Sep 09, due Sep 16**
> *(Note: Sep 07 is Labor Day — no class.)*

This unit answers the question students ask most often: **what actually happens when someone "builds an AI"?** The honest answer is that it is a pipeline of unglamorous steps, most of which involve data rather than algorithms. By the end of this unit you will have run that entire pipeline yourself in about fifteen lines of code.

## Learning Objectives

After completing this unit, you will be able to:

- Describe the **six stages** of the AI development lifecycle.
- Explain why **train/test splitting** is non-negotiable and what happens without it.
- Build, train, and evaluate a working model end to end.
- Read a **confusion matrix** and explain what accuracy hides.

## Part I — The AI Development Lifecycle

Every real AI system, from a small classifier to a frontier language model, passes through the same six stages.

| Stage | What happens | Share of effort |
| :--- | :--- | :---: |
| **1. Problem framing** | Decide what to predict and how success is measured | 10% |
| **2. Data collection** | Gather raw examples from sensors, records, text, users | 25% |
| **3. Preprocessing** | Clean, normalize, handle missing values, extract features | **35%** |
| **4. Training** | Fit model parameters to the data | 10% |
| **5. Evaluation** | Measure performance on data the model has never seen | 15% |
| **6. Deployment & monitoring** | Ship it, watch it, retrain as the world changes | 5% |

```{note}
Those percentages surprise people. **Stages 2 and 3 dominate.** The algorithm — the part that gets the headlines — is a small slice. Practitioners have a saying for this: *most of machine learning is data cleaning wearing a lab coat.* Unit 9 is devoted entirely to why.
```

### 1.1 The Golden Rule: Never Test on Training Data

A model that has memorized its training examples will score perfectly on them and may be useless on anything new. To detect this, we **hold data back**:

- **Training set** (typically 70–80%) → the model learns from this.
- **Test set** (typically 20–30%) → locked away, used exactly once, at the end.

> Testing a model on its training data is like grading an exam where the students were given the answer key. The score is real; it just measures nothing you care about.

The gap between training accuracy and test accuracy is the single most diagnostic number in machine learning. Unit 4 makes you watch that gap open up in real time.

### 1.2 Generalization

The goal is never to fit the data you have. It is to perform well on data you have not yet seen. This is **generalization**, and it is the entire point.

## ⚙️ Hands-On: Your First Complete AI System

This is the whole pipeline — all six stages — in one snippet. Copy it into Colab and run it.

```python
from sklearn.datasets import load_wine
from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import accuracy_score, confusion_matrix, classification_report

# --- STAGE 1: Problem framing -------------------------------------
# Task: given chemical measurements of a wine, predict which of three
# cultivars it came from. This is CLASSIFICATION (see Unit 2).

# --- STAGE 2: Data collection -------------------------------------
data = load_wine()
X, y = data.data, data.target        # X = features, y = labels
print(f"Examples: {X.shape[0]}   Features: {X.shape[1]}   Classes: {len(data.target_names)}")
print(f"Feature names: {list(data.feature_names[:4])} ...")

# --- STAGE 3: Preprocessing ---------------------------------------
# This dataset is already clean. Real data almost never is (Unit 9).

# --- STAGE 4: Training --------------------------------------------
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42, stratify=y
)
model = DecisionTreeClassifier(max_depth=3, random_state=42)
model.fit(X_train, y_train)          # <-- this line is "the AI learning"

# --- STAGE 5: Evaluation ------------------------------------------
train_acc = accuracy_score(y_train, model.predict(X_train))
test_acc  = accuracy_score(y_test,  model.predict(X_test))

print(f"\nTraining accuracy: {train_acc:.1%}")
print(f"Test accuracy:     {test_acc:.1%}   <-- the number that matters")
print(f"Gap:               {train_acc - test_acc:.1%}")

print("\nConfusion matrix (rows = actual, columns = predicted):")
print(confusion_matrix(y_test, model.predict(X_test)))
```

Notice how short Stage 4 is. **One line.** The mystique of "training an AI" is, mechanically, a single function call. Everything difficult lives on either side of it.

### Reading What You Just Produced

The **confusion matrix** is more informative than accuracy alone. Each row is the true class, each column what the model predicted. Values off the diagonal are mistakes — and *which* mistakes matter enormously in practice.

```python
import matplotlib.pyplot as plt
from sklearn.metrics import ConfusionMatrixDisplay

fig, ax = plt.subplots(figsize=(6, 5))
ConfusionMatrixDisplay.from_estimator(
    model, X_test, y_test,
    display_labels=data.target_names, cmap="Greens", ax=ax
)
ax.set_title("Where does the model get confused?")
plt.tight_layout()
plt.show()
```

> **Why accuracy lies.** Imagine a disease affecting 1 in 1000 people. A model that always predicts "healthy" is 99.9% accurate and medically worthless. It never catches a single case. Accuracy alone cannot tell you this — the confusion matrix can. In medicine, missing a sick patient (a false negative) and alarming a healthy one (a false positive) carry wildly different costs, and no single number captures both.

## ⚙️ Hands-On: Seeing Inside the Model

Decision trees have a rare property: you can read them. Most models cannot be inspected this directly, which is exactly the interpretability problem of Unit 10.

```python
from sklearn.tree import plot_tree

plt.figure(figsize=(16, 8))
plot_tree(model,
          feature_names=data.feature_names,
          class_names=data.target_names,
          filled=True, rounded=True, fontsize=9)
plt.title("The learned model, made visible", fontsize=13, weight="bold")
plt.show()
```

**Try changing it:**

1. Set `max_depth=1`. Accuracy drops — but is the model easier to trust? This is the **accuracy/interpretability trade-off** in miniature.
2. Set `max_depth=None` (unlimited). Watch the training accuracy hit 100% while the test accuracy does *not* improve. **You have just produced overfitting** — Unit 4 explains exactly what went wrong.
3. Change `random_state=42` to another number. The accuracy moves. What does that instability tell you about trusting a single reported number?

## 💡 Example: Rules vs. Learning, Revisited

In Unit 1 you wrote a rule-based spam filter that failed on *"Free coffee in the break room."* Here is the learned version.

```python
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes import MultinomialNB

emails = [
    "URGENT winner claim your free prize now click here",
    "congratulations you won a free cruise click here urgent",
    "free money click here limited offer act now urgent",
    "claim your prize winner free gift click",
    "Hi professor attaching my homework for unit three thanks",
    "Meeting moved to Wednesday at ten in Holmes 402",
    "Free coffee in the break room today, come by",
    "Reminder: capstone proposal due next week, see Canvas",
]
labels = [1, 1, 1, 1, 0, 0, 0, 0]      # 1 = spam, 0 = legitimate

vectorizer = CountVectorizer()
X_text = vectorizer.fit_transform(emails)

clf = MultinomialNB()
clf.fit(X_text, labels)

new_emails = [
    "free coffee available in the lounge this afternoon",
    "URGENT click here now to claim your free prize winner",
    "attaching the reading for next week",
]
preds = clf.predict(vectorizer.transform(new_emails))

print("LEARNED APPROACH")
for email, p in zip(new_emails, preds):
    tag = "SPAM" if p == 1 else "OK  "
    print(f"  {tag} | {email}")
```

The model was never told that "free" is ambiguous. It **inferred** from examples that *free* alongside *click* and *urgent* signals spam, while *free* alongside *coffee* does not. Nobody wrote that rule.

```{warning}
Eight training examples is absurdly few — this model is fragile and will fail on anything unusual. It is a demonstration, not a product. Real spam filters train on millions of messages and are retrained constantly, because spammers adapt. That adaptation problem is called **distribution shift**, and it is why Stage 6 (monitoring) never ends.
```

## 🧭 Reflection

> You just built a working classifier in fifteen lines. Did it feel like building intelligence?
> If a model can classify wines better than most humans while having no concept of wine, what exactly has been achieved?

**Connecting to HW3 (How AI Works — Summary & Questions, due Sep 16):** walk through the six stages for an AI system in a domain that interests you. Be specific about where the data would come from and what could go wrong at Stage 3.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 19. Pearson.
- Mitchell, M. (2020). _Artificial Intelligence: A Guide for Thinking Humans_, Part II. Penguin Books.
- Géron, A. (2022). _Hands-On Machine Learning with Scikit-Learn, Keras, and TensorFlow_, 3rd Ed., Ch. 2. O'Reilly.
- Dendritic Institute (2025). _AI Literacy Series — Module 3: How Machines Learn._
