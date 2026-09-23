# Appendix A: Python and Colab Primer

**Read this before Unit 3 if you have never run code before.** It takes about ten minutes and requires no installation. You will not be asked to *write* programs in this course — but you will run them, change small pieces, and interpret what comes out.

## Part I — Getting Started with Google Colab

**Google Colab** runs Python in your browser. Nothing to install, works on any laptop, free with a Google account.

### Step 1 — Open a notebook

Go to [colab.research.google.com](https://colab.research.google.com) and choose **New notebook**. Or click the 🚀 rocket icon at the top of any page in this book.

### Step 2 — Save your own copy

```{warning}
**Do this first, every time.** Click **File → Save a copy in Drive**. If you skip this, you are working in a temporary copy and everything you do disappears when you close the tab. This is the single most common way students lose their work.
```

### Step 3 — Run a cell

Click into a code cell and press **Shift + Enter** (or the ▶ play button). Try this:

```python
print("Hello from Colab")
2 + 2
```

The first run takes ten to twenty seconds while Colab starts a machine for you. Subsequent runs are instant.

### Step 4 — Know your shortcuts

| Action | Shortcut |
| :--- | :--- |
| Run cell, move to next | **Shift + Enter** |
| Run cell, stay put | **Ctrl + Enter** |
| Add cell below | **Ctrl + M** then **B** |
| Delete cell | **Ctrl + M** then **D** |
| Interrupt a stuck cell | **Runtime → Interrupt execution** |
| Start completely fresh | **Runtime → Restart session** |

## Part II — Just Enough Python

You need to *read* these six things, not write them from memory.

### Variables — names for values

```python
course = "Foundations of AI"      # text (a "string")
credits = 3                        # whole number (an "integer")
gpa = 3.75                         # decimal ("float")
enrolled = True                    # yes/no ("boolean")

print(course, credits, gpa, enrolled)
```

### Lists — ordered collections

```python
units = ["History", "Foundations", "How AI Works", "ML Paradigms"]

print(units[0])       # 'History'  -- counting starts at ZERO
print(units[-1])      # 'ML Paradigms'  -- negative counts from the end
print(len(units))     # 4
```

```{note}
**Counting from zero** trips up everyone at first. The first item is `[0]`, the second is `[1]`. Whenever code surprises you by being "off by one," this is usually why.
```

### Loops — do something repeatedly

```python
for unit in units:
    print(f"This week: {unit}")
```

The **indentation matters** in Python. The indented line is what repeats. This is not a style preference — it is the syntax.

### Functions — reusable operations

```python
def letter_grade(score):
    if score >= 93: return "A"
    elif score >= 90: return "A-"
    elif score >= 87: return "B+"
    elif score >= 83: return "B"
    else: return "see the syllabus"

print(letter_grade(94))
print(letter_grade(85))
```

### Libraries — borrowing other people's work

Almost all real code imports tools rather than building from scratch:

```python
import numpy as np              # numerical arrays
import pandas as pd             # tables of data
import matplotlib.pyplot as plt # charts
```

The `as np` part creates a short nickname. When you see `np.mean(...)`, it means "the `mean` function from numpy."

### Comments — notes for humans

```python
# Anything after a hash mark is ignored by Python.
# The code snippets in this book use comments heavily
# to explain what each section is doing. Read them.
```

## Part III — Reading Error Messages

Errors are normal and are not a sign you did something stupid. **Read the last line first** — it names the problem.

| Error | Usual meaning | Fix |
| :--- | :--- | :--- |
| `NameError: name 'x' is not defined` | Used something before creating it | Run the earlier cell first |
| `ModuleNotFoundError` | Library not installed | Add `!pip install <name>` |
| `IndentationError` | Spacing is inconsistent | Align the indented block |
| `SyntaxError` | Typo — often a missing `)` or `:` | Check the line above too |
| `KeyError` | Column name doesn't exist | Check spelling and capitalization |
| `ValueError: shapes not aligned` | Data dimensions mismatch | Print `.shape` to inspect |

```{important}
**The most common problem in Colab is running cells out of order.** Cells share memory, so a cell that worked yesterday may fail today because you skipped the one that defined its variables. When things get strange: **Runtime → Restart session**, then run every cell from the top.
```

### Using AI to debug

Per this course's AI policy, using an AI assistant on lab activities is **encouraged**. A productive approach:

> I'm getting this error in Python. Explain what it means in plain language and what I should check. Don't just give me corrected code — I want to understand it.
>
> [paste the full error message and the code that caused it]

Asking for an *explanation* rather than a *fix* is the difference between learning and copying. Your exams do not permit AI (see Appendix C).

## Part IV — What This Course Uses

Every snippet in this book runs on the **stock Colab environment**. These are already installed:

| Library | Purpose | Units |
| :--- | :--- | :--- |
| `numpy` | Numerical arrays and math | Throughout |
| `pandas` | Data tables | 6, 8, 9, 10, 12 |
| `matplotlib` | Charts and plots | Throughout |
| `scikit-learn` | Machine learning models | 3, 4, 5, 6, 9, 10 |
| `scipy` | Scientific computing | 13 |

Only Unit 7 requires an install (`transformers`), and the snippet includes the command.

## Part V — Verify Your Setup

Run this before Unit 3. If it prints without error, you are ready for the whole course.

```python
import sys
import numpy as np, pandas as pd, matplotlib, sklearn, scipy
import matplotlib.pyplot as plt

print(f"Python       {sys.version.split()[0]}")
print(f"numpy        {np.__version__}")
print(f"pandas       {pd.__version__}")
print(f"matplotlib   {matplotlib.__version__}")
print(f"scikit-learn {sklearn.__version__}")
print(f"scipy        {scipy.__version__}")

# A quick end-to-end check: train a model and draw a chart
from sklearn.datasets import load_iris
from sklearn.tree import DecisionTreeClassifier
from sklearn.model_selection import train_test_split

X, y = load_iris(return_X_y=True)
X_tr, X_te, y_tr, y_te = train_test_split(X, y, random_state=0)
acc = DecisionTreeClassifier(random_state=0).fit(X_tr, y_tr).score(X_te, y_te)

plt.figure(figsize=(5, 3))
plt.bar(["accuracy"], [acc], color="seagreen")
plt.ylim(0, 1); plt.title(f"Setup OK — model accuracy {acc:.0%}")
plt.tight_layout(); plt.show()

print("\n✅ Everything works. You are ready for Unit 3.")
```

## Alternatives to Colab

You are welcome to use any Python environment you prefer:

| Option | Notes |
| :--- | :--- |
| **Google Colab** | Recommended. Nothing to install. |
| **Anaconda / Jupyter** | Local install; good if you want offline work |
| **VS Code** + Python extension | Popular with developers |
| **Kaggle Notebooks** | Similar to Colab, also free |
| **PyCharm** | Full IDE; more setup |

Every snippet in this book is plain Python and runs anywhere. The only Colab-specific line in the entire book is the `!pip install` in Unit 7 — outside Colab, run `pip install transformers` in your terminal instead.
