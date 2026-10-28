# Unit 10: Documents and Texts

```
Session 16 | Oct 19 | HW10 (Office Portfolio) assigned Oct 19, due Oct 25
```

Part IV covers the three document types that consume most professional working hours: text, slides, and spreadsheets. By this point in the course you already know how to write a good prompt. This unit is not about prompting. It is about what happens **after** the prompt: whether the output has the structure you asked for, whether it says only what the source says, and whether you can prove either of those things.

Every activity in this unit follows the same pattern. AI produces something that **looks** finished. Your job is to check whether it **is** finished, using a test that does not depend on how it looks. A document can look like it has headings and have none. A summary can read well and contain a decision nobody made. A sentiment label can come with a fluent explanation that has nothing to do with how the label was produced.

## Learning Objectives

After completing this unit, you will be able to:

- Use AI to **interrogate** a long document rather than merely summarize it.
- Generate code documentation and verify it against what the code actually does, not what its names suggest.
- Use AI to restructure a Word document with real heading styles, captions, a table of contents, and tracked changes, and verify that the structure is real.
- Convert a Word document into a compilable LaTeX project and audit the conversion.
- Compare an LLM and a Naive Bayes classifier on the same sentiment task, and distinguish a **faithful** explanation from a **plausible** one.
- Turn rough notes into meeting minutes that flag gaps instead of filling them.
- Identify what must never be pasted into a third-party tool.

## Part I — Working with Documents

### 1.1 What AI Does Well with Text, and What It Does Not

| Genuinely useful | Requires care | Do not delegate |
| :--- | :--- | :--- |
| Restructuring your rough draft | Summarizing something you have not read | Deciding what the document should claim |
| Adjusting register for an audience | Any output containing numbers or citations | Judgments about people |
| Converting between formats | Technical accuracy on unfamiliar material | Anything you will sign your name to unread |
| Finding what a long document says about X | Tone in emotionally loaded situations | Commitments on behalf of other people |

The organizing principle: **AI is strong at transformation and weak at commitment.** Give it your thinking and ask it to reshape that, and it performs well. Ask it to supply the thinking, and you get the most statistically typical version of the document, which is by definition unremarkable, and possibly wrong.

Every task in this unit is a transformation: code into documentation, a flat document into a structured one, Word into LaTeX, text into sentiment labels, notes into minutes. That is why AI is useful here. It is also why each task needs a check that the transformation preserved what it was supposed to preserve.

### 1.2 Four Things AI Can Do with a Document

AI is extraordinarily useful for working with longer-form written content, such as reports, articles, meeting notes, policy documents, proposals, and more. To illustrate, consider the paper about the use of Transfer Learning in Bioinspired Algorithms authored by Rita Xavier and Dr. Leandro de Castro, published in the Artificial Intelligence Reviews: [Download the paper](../Data/TLBIA_AIR.pdf). Use the paper on Transfer Learning or any paper from your capstone domain.

**Summarizing.** The most familiar use, and the least informative.

```
Summarize the attached paper in five bullet points for a reader who works in industry and has no background in natural computing.
Focus on what the authors claim, not on how the paper is organized.
After the bullets, list any claim in your summary that is not stated explicitly in the paper but that you inferred.
```

**Targeted extraction.** Far more useful, because the answer can be checked.

```
Answer the three questions below using only the attached paper.
For each answer, give the section number and quote the sentence that supports it.
If the paper does not answer a question, write "Not addressed in the paper" and stop.

1. How do the authors define transfer learning?
2. Which families of bioinspired algorithms do they review?
3. What open problems do they identify for future research?
```

**Explaining specialist language.** Useful when the stakes of misreading are real.

```
Explain the paragraph below in plain English for a tenant with no legal background.
Then list every obligation it places on the tenant as a separate bullet, and state what would happen if the tenant ignored each one.

"Notwithstanding any other provision of this Agreement, Tenant shall indemnify, defend, and hold harmless Landlord and its respective officers, directors, employees, agents, successors, and assigns from and against any and all claims, damages, losses, costs, and expenses, including reasonable attorneys' fees, arising out of or related to Tenant's use or occupancy of the Premises, any breach or default by Tenant under this Agreement, or any act or omission of Tenant or Tenant's agents, employees, contractors, or invitees."
```

**Turning notes into documents.** Covered in depth in Part VI, because it is where AI most often invents things.

### 1.3 Interrogate, Do Not Summarize

Summaries are the least useful thing to ask for, because they discard exactly the specificity you needed and you have no way to tell what was dropped. Better patterns:

- **Targeted extraction.** *"What does this specification say about error handling? Quote the relevant passages and give section numbers. If it says nothing, say so."*
- **Contradiction hunting.** *"Identify places where this document contradicts itself or leaves a requirement ambiguous."*
- **Question answering with refusal.** *"Answer only from this document. If the answer is not present, say 'not addressed' and stop."*
- **Comparison.** *"What changed between version 1 and version 2 that would require me to change what I do?"*

Each of these produces something checkable. A summary does not.

### 1.4 Tools Built for Documents

General-purpose assistants (Claude, ChatGPT, Gemini, Copilot) handle most of this unit. Two specialized tools are worth knowing because they change how you work with **many** documents at once.

| Tool | Best for | What makes it different |
| :--- | :--- | :--- |
| [NotebookLM](https://notebooklm.google.com) | Exploring a set of your own sources | Answers only from the sources you upload, with citations to the passage. Generates audio overviews, mind maps, quizzes, and reports from the same sources. |
| [SciSpace](https://scispace.com) | Reading academic papers | Explains equations, tables, and methods in plain language; searches a large database of papers; generates citations. |
| Microsoft Copilot in Word | Editing inside Word | Works on the open document directly, including styles and comments. |
| Grammarly / Wordtune | Sentence-level rewriting | Real-time suggestions as you type; does not restructure a document. |


## Part II — Documenting Code

Code documentation is the textbook case of a transformation task: the information is all in the code, and the job is to make it readable. AI does this well. It also has a specific, predictable failure: it documents what the **names** suggest rather than what the **logic** does. A function called `average_score` will be documented as computing an average, whether or not it does.

This is the same principle you have applied since Unit 1: reason from the internal mechanism, not the surface.

```{note}
You do not need to write code for this part. You need to read about fifteen lines of Python carefully, which everyone in this course can do after Units 3 to 5.
```

### 2.1 The Prompt

```
You are documenting Python functions for a colleague who knows the language but has never seen this code.

For each function below, write:
1. A docstring with: Purpose, Parameters (with types and units), Returns, Raises.
2. Inline comments on any line whose purpose is not obvious.
3. A "Caveats" list naming every assumption the function makes about its input: units, missing values, duplicates, ordering, empty input.

Rules: Document only behavior visible in the code.
Where a description comes from a function or variable name rather than from the logic, mark it [INFERRED FROM NAME].
Where the behavior on an edge case is unclear, mark it [UNCLEAR] rather than guessing.
Then, for each function, give one example call and its exact output.

def bmi(weight, height):
    return round(weight / (height ** 2), 1)

def remove_duplicates(records):
    seen = {}
    for r in records:
        key = r["email"].strip().lower()
        seen[key] = r
    return list(seen.values())

def average_score(scores):
    total = 0
    count = 0
    for s in scores:
        if s:
            total += s
            count += 1
    return total / count
```

### 2.2 What to Check

Each function hides something that a name-based reading will miss. Do not read the answers below until your group has checked the AI output yourselves.

```{admonition} What the code actually does
:class: dropdown

**`bmi`** assumes metric units (kilograms and meters). Called with pounds and inches, `bmi(154, 69)` returns `0.0`, silently, instead of an error.

**`remove_duplicates`** treats `"Ana@x.com"` and `" ana@x.com "` as the same person, and keeps the **last** occurrence, not the first. The returned record still contains the original, unnormalized email.

**`average_score`** skips every value for which `if s` is false. That includes `None` (probably intended) and `0` (probably not). A student with scores `[90, 0, 80, None, 70]` receives an average of `80.0`. The true average of the four numeric scores is `60.0`. If every score is zero, the function crashes with `ZeroDivisionError`.
```

**Verify:** run each example call the AI gave you in Colab. Any example whose stated output does not match the real output means the documentation was written from the names.

The `average_score` case is the important one. Documentation that says "returns the average of the scores" is not a small inaccuracy. It hides a behavior that changes students' grades, and the next person to maintain the code will trust the docstring instead of reading the loop.

### Using Context Engineering
To avoid the problems described above, it is important to provide the LLM with the necessary software engineering context.

```
You are a Python code documentation specialist.
Your task is to document Python functions for a colleague who understands Python but has never seen this code.

**Your primary objective is behavioral accuracy, not interpretation of the programmer's intentions.**

For each function, produce:

1. **Docstring**, containing:
   - **Purpose:** Describe precisely what the function computes or performs, based exclusively on its executable logic.
   - **Parameters:** Describe each parameter's required operations, supported types or type constraints, and units (only when explicitly established by the code).
   - **Returns:** Describe the actual returned value, its type when determinable, and how it is constructed.
   - **Raises:** Identify exceptions that can be established from the code, including the conditions that trigger them.

2. **Inline comments:** Add comments only where they clarify non-obvious operations, control flow, side effects, overwriting, filtering, or ordering behavior. Do not merely restate Python syntax.

3. **Caveats:** Explicitly examine:
   - Units and measurement assumptions.
   - Missing values, `None`, `NaN`, and falsy values.
   - Duplicates and repeated values.
   - Input and output ordering.
   - Empty inputs and zero denominators.
   - Type constraints and implicit conversions.
   - Boundary conditions and unvalidated assumptions.
   
   Distinguish between assumptions the implementation actually makes and conditions it does not validate. Mark categories that do not apply as "Not applicable."

4. **Example:** Provide one executable example call and its exact output, consistent with the original implementation.

### Strict rules

**Rule 1 — Executable code is the source of truth.**
Derive every behavioral claim from expressions, operators, branches, loops, indexing, assignments, built-in functions, and return statements.
Do not infer functionality from the function's name, parameter names, variable names, comments, or presumed application domain.

**Rule 2 — Separate observed behavior from inferred intent.**
If you include a semantic interpretation supported only by a name or contextual clue, explicitly label it `[INFERRED FROM NAME]`.
Prefer omitting such interpretations entirely when they do not improve understanding of the actual implementation.

**Rule 3 — Do not invent guarantees or requirements.**
Do not assume input units, data validation, missing-value handling, accepted ranges, sorting, uniqueness, or error handling unless the implementation establishes them.
Do not equate an operation's effect with its presumed intent.

**Rule 4 — Be precise about types and exceptions.**
Distinguish between types explicitly enforced by the code and types that merely support the required operations.
Identify exceptions and output types only when they follow from the implementation and stated input conditions.
Account for relevant Python language and standard-library behavior.

**Rule 5 — Mark genuine uncertainty.**
Use `[UNCLEAR]` when behavior depends on unspecified custom types, external state, dependencies, or Python versions.
Do not mark behavior unclear when it is determinable from standard Python semantics.

**Rule 6 — Preserve implementation.**
Do not change executable statements, variable names, expressions, conditionals, or return values. Add only docstrings and comments.
If you identify a potential defect, document its observable consequences under Caveats rather than silently fixing it.

**Rule 7 — Verify every claim.**
Before finalizing each function:
- Trace the executable operations independently of all identifiers.
- Check that every statement in the documentation is supported by those operations.
- Test mentally or execute the example using the original implementation.
- Confirm that the documented edge cases, exception conditions, and return behavior match the code.
- Remove or qualify any claim that cannot be justified by the implementation.

### Output format

For each function, provide:
1. The original function with its new docstring and inline comments.
2. A Caveats section.
3. One example call with its exact output.

Do not propose improvements or refactor the code unless explicitly requested.

### Code to document

python
def bmi(weight, height):
    return round(weight / (height ** 2), 1)

def remove_duplicates(records):
    seen = {}
    for r in records:
        key = r["email"].strip().lower()
        seen[key] = r
    return list(seen.values())

def average_score(scores):
    total = 0
    count = 0
    for s in scores:
        if s:
            total += s
            count += 1
    return total / count
```

## Part III — Structuring a Word Document

Most Word documents only **look** structured. Headings are bold 14-point text. Captions are typed by hand as "Figure 3". The table of contents, if there is one, was typed manually and is already out of date. Word cannot see any of that structure, so nothing that depends on it works: the Navigation Pane, automatic numbering, cross-references, the table of figures, and accessibility tools that read headings aloud.

AI can repair this, and it can do it with **tracked changes**, so that every edit is visible and reversible.

### 3.1 The Prompt

Attach the Word document below to an assistant that can read and write files (Claude, ChatGPT, or Copilot in Word), then runthe following prompt:
[Word document](../Data/word_document_editing_test.docx)

```
I have attached a Word document. Return a revised .docx file with the following changes.

STRUCTURE
1. Apply Word's built-in Heading 1, Heading 2, and Heading 3 styles to every section title, based on the document's logical hierarchy. Do not just make text bold or larger.
2. Replace every typed figure and table label with a real Word caption (References > Insert Caption), using automatic numbering. Figure captions go below figures; table captions go above tables.
3. Replace every typed reference to a figure or table in the body text (for example, "see Figure 2") with a cross-reference field that updates automatically.
4. Insert an automatic Table of Contents after the title, built from the heading styles, and an automatic List of Figures and List of Tables after it.

EDITING
5. With Track Changes turned on and the author set to "AI Reviewer", correct grammar, spelling, and clarity problems. Do not change the meaning of any sentence or any technical term.
6. Add a Word comment (not a tracked change) wherever a claim lacks a source, a sentence is ambiguous, or a number seems inconsistent with another part of the document. Do not fix these yourself.

After the file, give me a short report: how many headings you styled at each level, how many captions you created, how many tracked changes and how many comments you added.
```

### 3.2 What to Check

Open the returned file in Word and run these four tests. None of them can be passed by formatting that only looks right.

| Test | How | Passes if |
| :--- | :--- | :--- |
| Headings are real | View > Navigation Pane | Every section appears, at the right level |
| Captions are real | References > Insert Table of Figures | All figures are listed with correct numbers |
| TOC is live | Right-click the TOC > Update Field > Update entire table | Page numbers fill in and nothing disappears |
| Edits are tracked | Review > Reviewing Pane | Changes are listed under "AI Reviewer" and each can be accepted or rejected |

Then compare the AI's report in step 6 with what you count. Reports of work done are themselves AI output, and they are wrong more often than the work.

```{note}
A TOC generated outside Word sometimes opens showing "Right-click to update field" or empty page numbers. This is expected: page numbers depend on layout, which only Word computes. Updating the field fixes it. If the update produces an empty TOC, the headings were not real, which is test 1 failing.
```

## Part IV — From Word to LaTeX

LaTeX is the standard format for technical publishing in engineering, mathematics, and the sciences, and [Overleaf](https://www.overleaf.com) is the browser-based editor most researchers use. Converting a Word document into LaTeX by hand is slow and mechanical, which makes it a good candidate for AI. It is also full of places where a conversion can silently go wrong.

The good news is that LaTeX gives you an objective test: **either it compiles or it does not**, and either the cross-references resolve or they show `??`.

Use the structured document you produced in Part III as the input. The work you did there makes this conversion much easier, because real headings and captions map directly onto `\section` and `\caption`.

### 4.1 The Prompt

```
I have attached a Word document. Convert it into a LaTeX project that compiles in Overleaf with pdfLaTeX, without errors.

Deliver a single .zip file that I can upload to Overleaf with New Project > Upload Project, containing:
1. main.tex using the article class.
2. refs.bib containing every reference in the document as a BibTeX entry, cited in the text with \cite.
3. A folder named figures/ containing every image extracted from the Word document, with the exact file names main.tex expects.

Before giving me the .zip, confirm that every file named in an \includegraphics command exists in the figures/ folder.

Conversion rules:
- Word headings become \section, \subsection, and \subsubsection.
- Every figure and table gets a \caption and a \label, and every reference to them in the text becomes \ref. No figure or table number may be typed by hand.
- Every equation becomes a numbered equation environment with a \label. Reproduce each equation exactly. Do not simplify or rewrite any mathematics.
- Tables use the booktabs package.
- Use \tableofcontents after the title.
- Do not add, remove, or reword any sentence.

After the files, list anything in the Word document you could not convert faithfully (for example, text boxes, merged table cells, colored text, footnotes), and what you did with each.
```

### 4.2 What to Check

1. **Compile.** In Overleaf, choose New Project > Upload Project, select the .zip, and compile. Open the `figures/` folder and confirm that it contains every image from the Word document. Every error message is a conversion failure. Paste the error back into the assistant and ask for the fix, but record each one.
2. **Cross-references.** Search the PDF for `??`. Each one is a `\ref` or `\cite` that points at nothing.
3. **Counts.** Number of sections, figures, tables, equations, and references in the PDF must match the Word document exactly.
4. **Equations.** Compare every equation symbol by symbol with the original. This is where conversions fail most quietly: a missing exponent or a swapped subscript compiles perfectly.
5. **Text.** Pick three paragraphs at random and compare them word for word with the original.

```{note}
Some assistants cannot create .zip files, especially on free plans. In that case, ask for main.tex and refs.bib as separate files, and extract the images yourself: make a copy of the .docx, rename it to .zip, open it, and copy the files from the `word/media` folder into a `figures/` folder in Overleaf. Check that their names match the `\includegraphics` commands in main.tex.
```

```{tip}
**Going further.** [Pandoc](https://pandoc.org), a free conversion tool, can also convert Word to LaTeX, deterministically and with no AI involved. In Colab, upload your .docx and run `!pandoc input.docx -s -o pandoc.tex --extract-media=.` Compare the two outputs: which handles equations better, which handles references better, and which makes errors you would not notice without checking? The point is not that one tool wins. A deterministic tool always makes the same mistakes; an LLM may make different ones each time.
```

## Part V — Sentiment Analysis: An LLM and Naive Bayes

Sentiment analysis classifies text as positive, negative, or neutral. It is one of the oldest applications of machine learning to text, and you have already met the classical approach: in Unit 3, Multinomial Naive Bayes separated spam from legitimate messages by learning how often each word appears in each class. An LLM can perform the same task with no training data at all.

This part runs both on the same document and compares them on three things: accuracy, consistency, and **the honesty of their explanations**.

### 5.1 The LLM Side

Send the prompt below to an assistant, in a new conversation.

```
Classify the sentiment of each of the eight sentences below, which come from one hotel guest's review, as POSITIVE, NEGATIVE, or MIXED.

For each sentence, explain step by step how you reached your label: which words or phrases you relied on, how you handled negation, sarcasm, or contrast, and how confident you are (low, medium, high).

Then give an overall sentiment for the whole review and explain how you combined the sentences.

1. The room was clean and the staff were friendly.
2. The breakfast was not bad at all.
3. Oh great, another hour waiting for the elevator.
4. The location is perfect but the walls are paper thin.
5. Nothing about the spa was disappointing.
6. I would not recommend the restaurant.
7. The front desk solved my problem quickly.
8. Check-out took forever.
```

Then run these three follow-ups in the same conversation:

```
Give me a numeric weight for every word in sentence 2, showing how much each word pushed your decision toward positive or negative. The weights must add up to your final decision.
```

```
Is the explanation you gave for sentence 3 a description of the computation you actually performed to produce the label, or an account written after the label was produced? Answer directly.
```

Finally, open **two new conversations** and send the original prompt again in each. Record the labels from all three runs.

### 5.2 The Naive Bayes Side

## ⚙️ Hands-On: A Classifier That Shows Its Work

This cell trains a Multinomial Naive Bayes classifier on 42 short labeled reviews from several domains, then classifies the same eight sentences. For each sentence it prints the contribution of every word: the difference between the log-probability of the word in positive reviews and in negative reviews. Those numbers are not an explanation of the decision. They **are** the decision.

```python
import numpy as np
import pandas as pd
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes import MultinomialNB

# --- TRAINING DATA: short labeled reviews from several domains ------------
train = [
    ("The food was delicious and the staff were friendly", "pos"),
    ("Great service, we will definitely come back", "pos"),
    ("The room was clean, quiet, and comfortable", "pos"),
    ("Excellent course, the professor explains everything clearly", "pos"),
    ("I love this app, it is fast and easy to use", "pos"),
    ("The nurse was kind and helpful during my visit", "pos"),
    ("Wonderful experience from start to finish", "pos"),
    ("The battery life is amazing and the screen is beautiful", "pos"),
    ("Helpful support team, my problem was solved quickly", "pos"),
    ("The views were stunning and the guide was great", "pos"),
    ("Fresh ingredients and generous portions, highly recommend", "pos"),
    ("Smooth delivery, the package arrived early", "pos"),
    ("The lectures were engaging and well organized", "pos"),
    ("Comfortable seats and a pleasant flight", "pos"),
    ("Best purchase I made this year, works perfectly", "pos"),
    ("The pharmacist was patient and answered all my questions", "pos"),
    ("Friendly neighbors and a safe, quiet street", "pos"),
    ("The update made the app faster and more reliable", "pos"),
    ("Great value for the price, very happy", "pos"),
    ("The checkout was quick and the staff were polite", "pos"),
    ("The location was perfect and the music was lovely", "pos"),
    ("The food was cold and the staff were rude", "neg"),
    ("Terrible service, we will never come back", "neg"),
    ("The room was dirty, noisy, and uncomfortable", "neg"),
    ("Boring course, the professor explains nothing clearly", "neg"),
    ("I hate this app, it is slow and crashes constantly", "neg"),
    ("The wait was long and nobody helped me", "neg"),
    ("Awful experience from start to finish", "neg"),
    ("The battery dies quickly and the screen is dim", "neg"),
    ("Support never answered, my problem is still not solved", "neg"),
    ("The tour was rushed and the guide was late", "neg"),
    ("Stale bread and tiny portions, overpriced", "neg"),
    ("The package arrived late and damaged", "neg"),
    ("The lectures were confusing and disorganized", "neg"),
    ("Cramped seats and a delayed flight", "neg"),
    ("Worst purchase I made this year, broke in a week", "neg"),
    ("The billing office was unhelpful and rude", "neg"),
    ("Loud neighbors and a dangerous street", "neg"),
    ("The update made the app slower and buggy", "neg"),
    ("Poor value for the price, very disappointed", "neg"),
    ("The checkout was slow and the staff were impatient", "neg"),
    ("The coffee was bad and the music was too loud", "neg"),
]
texts, labels = zip(*train)

# --- THE DOCUMENT: one guest review, analyzed sentence by sentence --------
document = [
    "The room was clean and the staff were friendly.",
    "The breakfast was not bad at all.",
    "Oh great, another hour waiting for the elevator.",
    "The location is perfect but the walls are paper thin.",
    "Nothing about the spa was disappointing.",
    "I would not recommend the restaurant.",
    "The front desk solved my problem quickly.",
    "Check-out took forever.",
]

vec = CountVectorizer()
X = vec.fit_transform(texts)
model = MultinomialNB(alpha=1.0).fit(X, labels)        # EDIT: smoothing

# log P(word | pos) - log P(word | neg): the model's entire "reasoning"
pos_i = list(model.classes_).index("pos")
neg_i = list(model.classes_).index("neg")
weight = model.feature_log_prob_[pos_i] - model.feature_log_prob_[neg_i]
vocab = vec.vocabulary_

rows = []
for sentence in document:
    tokens = vec.build_analyzer()(sentence)
    known = [t for t in tokens if t in vocab]
    unknown = [t for t in tokens if t not in vocab]
    pred = model.predict(vec.transform([sentence]))[0]
    p_pos = model.predict_proba(vec.transform([sentence]))[0][pos_i]
    rows.append({"sentence": sentence, "NB": pred, "P(pos)": round(p_pos, 2)})

    print(f"\n{sentence}")
    for t in known:
        w = weight[vocab[t]]
        print(f"   {t:<14}{w:+.2f}  {'(pushes POS)' if w > 0 else '(pushes NEG)' if w < 0 else ''}")
    if unknown:
        print(f"   ignored (never seen in training): {', '.join(unknown)}")
    print(f"   --> NB says {pred.upper()}  (P(pos) = {p_pos:.2f})")

print("\n" + "=" * 70)
table = pd.DataFrame(rows)
table["LLM"] = ""          # fill in from your LLM run
table["Your label"] = ""   # fill in by reading the sentence yourself
print(table.to_string(index=False))
```

**Try changing it:**

1. Add `("The breakfast was not bad, actually quite good", "pos")` to the training data and rerun. Sentence 2 is now positive. Look at the weights to see **why**: did the model learn that "not bad" is positive, or that the word `breakfast` is? Now check sentence 6. Fixing one sentence broke another. Why, when Naive Bayes scores every word independently of its neighbors?
2. Look at sentence 8, *Check-out took forever.* Every word is ignored, and the model still returns a label with P(pos) = 0.50. Where did that label come from? (Hint: in Try 1 the same sentence flips to positive, and no word in it changed. Count the training sentences in each class.) Add `("The line took forever and nobody apologized", "neg")` and rerun.
3. Change `alpha=1.0` to `alpha=0.01`, then to `alpha=10`. Watch the word weights and the probabilities. Which setting makes the model most confident, and is it also the most correct?

### 5.3 Comparing the Two

Fill in the `LLM` and `Your label` columns, then discuss.

| | Naive Bayes | LLM |
| :--- | :--- | :--- |
| Training data needed | Labeled examples | None |
| Handles negation and sarcasm | No, by design | Usually |
| Same input, same output | Always | Not guaranteed |
| Explanation | **Faithful**: the printed weights are the computation | **Plausible**: written in language, not read off the computation |
| Cost per 1 million sentences | Seconds on a laptop | Considerable time and money |
| Fails how | Visibly: you can see which word misled it | Invisibly: a wrong label arrives with a fluent reason |

On these eight sentences, Naive Bayes gets the clear cases right and fails on negation (sentences 2 and 5), sarcasm (sentence 3), and contrast (sentence 4, which it calls positive because it never saw `but`, `walls`, or `thin`). The LLM will very likely do better on all four. That is not the interesting result.

The interesting result is the explanations. When you asked the LLM for word weights that "add up to its decision", it gave you numbers. Those numbers were generated the same way as the rest of its answer: as plausible text. They were not read from the model's internal computation, because a transformer does not compute sentiment by adding word weights. Naive Bayes does, which is why its explanation can be trusted even when its answer is wrong.

```{important}
A **faithful** explanation describes the computation that actually produced the answer. A **plausible** explanation is one that would justify the answer. An LLM's step-by-step reasoning is often useful, and often correct, but it is a plausible explanation by construction. Never treat it as an audit trail.
```

**The two approaches complement each other.** A common professional pattern uses both: an LLM labels a few hundred examples, a human checks and corrects a sample of those labels, and a fast, cheap, transparent classifier such as Naive Bayes is trained on them and run on millions of documents. The LLM supplies language understanding where it is needed; the classical model supplies speed, consistency, and an explanation you can inspect.

## Part VI — Meeting Minutes from Rough Notes

Turning notes into minutes is the task where AI is most tempting and most dangerous, because minutes become the official record of who agreed to what. An assistant asked to produce "professional minutes" will produce minutes that look complete. Notes are never complete. The gap is filled with invention: an owner assigned to an action nobody volunteered for, a deadline nobody set, a decision that was only discussed.

### 6.1 The Prompt

```
You are the secretary of a university curriculum committee. Turn the rough notes below into formal meeting minutes.

Format:
- Header: meeting name, date, time, location, chair, attendees, absent.
- Agenda items, numbered. Under each: Discussion (two to four sentences), Decision (if one was made), and Action items.
- A final Action Items table with columns: Action | Owner | Deadline | Source line in notes.
- Next meeting.

Rules, and these are strict:
- Use only information in the notes. Do not add attendees, decisions, owners, or deadlines.
- Separate decisions from discussion. If the notes show something was discussed but not decided, write "No decision recorded."
- Wherever a required field is missing, write [NOT IN NOTES]. Do not fill it from context.
- For every action item, quote the line from the notes it came from.
- After the minutes, list every place where the notes are ambiguous and what question I should ask the attendees.

NOTES:
curriculum cmte — Oct 9, 2pm, Holmes 214
there: LdC (chair), Maria S, Tom R, Priya, Dave (left early ~2:40)
Jenna out sick

1. AI elective - CAI 4XXX
- maria: enrollment cap 35? dept says 40 ok
- tom worried about lab capacity, only 30 seats in 402
- agreed to go w/ 30 for spring, revisit fall
- need syllabus draft - priya?

2. capstone rubric
- current rubric too vague on "feasibility"
- dave suggested adding cost estimate criterion
- discussion, some disagreement, tom prefers keeping weights same
- LdC to circulate revised version

3. accreditation visit
- ABET site visit spring - date TBD
- need course binders updated
- everyone to check their own courses before break

next mtg - 2 wks? check w/ Jenna
```

### 6.2 What to Check

Using the minutes the AI returned, find:

- **Item 1:** Does the Action Items table name Priya as owner of the syllabus draft? The notes only say "priya?", a question, not an assignment.
- **Item 2:** Is the capstone rubric change recorded as a decision? The notes record a disagreement and no decision. Does the revised version from LdC have a deadline? The notes give none.
- **Item 3:** What deadline did the AI give for checking the course binders? "Before break" is in the notes; a specific date is not.
- **Attendance:** Is Dave's early departure recorded? Is Jenna listed as absent, and with what reason? Should the reason be in the minutes at all?
- **Next meeting:** Did the AI give a date?

Every invented owner, decision, or date is a commitment someone did not make, now in the official record. The `Source line in notes` column is what lets you catch these: any action item whose quoted source line does not actually say what the table says is an invention.

```{note}
This exercise works with your own notes too. Take notes during any meeting, club event, or group project session this week, and run the same prompt. Real notes are messier than the sample and produce more interesting gaps.
```

## 🗓️ Session Plan: In Class and At Home

| Where | Time | Activity |
| :--- | :--- | :--- |
| In class | 0–10 min | Part I: interrogate a paper with targeted extraction; compare with a plain summary |
| In class | 10–25 min | Part II: document the three functions, then run the example calls in Colab |
| In class | 25–55 min | Part V: LLM prompt, three-run consistency check, Naive Bayes cell, comparison table |
| In class | 55–70 min | Part VI: meeting minutes from the sample notes, gap audit |
| In class | 70–75 min | Debrief: which failure surprised you, and where would it matter in your capstone? |
| At home | | Part I: NotebookLM with the two Dietary Guidelines documents |
| At home | | Part III: structure a Word document with tracked changes (HW10) |
| At home | | Part IV: convert it to LaTeX and compile in Overleaf (HW10) |

## Part VII — Confidentiality

This is the responsibility beat for Unit 10, and it is the one most likely to matter to you during an internship.

Documents are the highest-risk category for accidental disclosure, because the useful thing to do with a document is paste or upload the whole thing. Every activity in this unit involves uploading a file.

**Before uploading any document, ask:**

1. **Whose document is it?** Employer, client, and university documents are frequently not yours to share, regardless of whether you wrote them.
2. **Does it name people?** Names plus any attached fact (performance, health, grades, conduct) is personal data. Meeting minutes almost always qualify.
3. **Does it contain credentials?** Code, configuration files, and error logs routinely embed passwords, keys, internal paths, and hostnames.
4. **What is the retention policy of the tool I am about to use?** Institutional accounts and personal accounts often have different policies.

The safe habit is **redact before upload**: replace names, keys, and identifiers with placeholders. It takes thirty seconds and it converts an irreversible mistake into a non-event.

## 💡 Example: A Summary That Cost a Sprint

An engineer asked an assistant to summarize a 60-page client specification and reported to the team that authentication was "OAuth 2.0 with standard scopes." The specification did say that, in section 4. Section 11, in a table the assistant never surfaced, added a mandatory client-certificate requirement.

The summary was not false. It was **incomplete in a way the summary itself could not disclose**, because a summary's job is to omit and it has no way to tell you whether what it omitted mattered.

Targeted extraction (*"list every authentication requirement in this document with section numbers"*) would have surfaced both. The difference is not the model. It is the question.

## 🧭 Reflection

> In Part V, Naive Bayes was wrong on four sentences out of eight, and you could see exactly why for each one. The LLM was probably right on more of them, and you could not see why for any of them.
>
> In your capstone domain, which would you rather defend in front of the person affected by a wrong decision: a model that is more often right, or a model whose mistakes you can explain? Is there a domain where the answer flips?

**Connecting to HW10 (Office Portfolio, Units 10–12):** this homework spans all three Part IV units and is due Oct 21. For Unit 10, submit **two documents**, both built from the same source: a Word report you wrote for any course (at least four pages, two figures, one table, one equation, and three references).

1. **The structured Word document (Part III):** the original .docx, the AI-revised .docx with tracked changes and comments, and a screenshot of each of the four tests in Section 3.2.
2. **The LaTeX version (Part IV):** your Overleaf project (shared link or .zip) and the compiled PDF.

For each document include the prompt you used, the raw output, every error you found and fixed, and a note on what you changed and why. The errors you found are what is graded, not the polish of the final document.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 23–24. Pearson.
- Mitchell, M. (2020). _Artificial Intelligence: A Guide for Thinking Humans_, Part IV ("Artificial Intelligence Meets Natural Language"). Penguin.
- Jurafsky, D., & Martin, J. H. (2025). _Speech and Language Processing_, 3rd Ed. draft, chapter on Naive Bayes, text classification, and sentiment. [https://web.stanford.edu/~jurafsky/slp3/]
- Turpin, M., Michael, J., Perez, E., & Bowman, S. R. (2023). Language Models Don't Always Say What They Think: Unfaithful Explanations in Chain-of-Thought Prompting. _Advances in Neural Information Processing Systems 36_.
- Overleaf. _Learn LaTeX in 30 Minutes._ [https://www.overleaf.com/learn/latex/Learn_LaTeX_in_30_minutes]
- Dendritic Institute (2026). _AI for Everyday Life — Module 2, Part I: AI for Emails and Written Communication._ [https://lndecastro.github.io/AI4EL/Module2_Daily_Assistant.html]
