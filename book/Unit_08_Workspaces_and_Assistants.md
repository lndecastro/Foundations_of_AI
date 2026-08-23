# Unit 8: AI Workspaces and Assistants

> **Sessions 13–14** · Sep 30, Oct 05 · **HW8 assigned Oct 05, due Oct 12**

Unit 7 ended on context engineering: the observation that what the model can *see* matters more than what you *type*. This unit is about the tools built to control that, and about the shift they represent — from asking an assistant questions to **configuring** one.

The distinction is worth stating early. A chat window is a stateless request. A workspace is a persistent environment with your documents, your instructions, and your conventions already loaded. Moving from the first to the second is the single largest productivity change most people make with these tools, and it costs about twenty minutes of setup.

## Learning Objectives

After completing this unit, you will be able to:

- Distinguish a **conversation**, a **workspace**, and a **configured assistant**, and choose between them.
- Explain **grounding** and how retrieval reduces (but does not eliminate) fabrication.
- Configure a working assistant with persistent instructions and a document base.
- Identify what must **never** be placed in a third-party workspace, and why.

## Part I — Three Levels of Working with AI

### 1.1 The Conversation

A single thread. Everything the model knows comes from what you typed in that thread. Nothing persists.

**Good for:** one-off questions, quick drafts, anything you will not repeat.
**Bad for:** anything where you find yourself re-explaining your project every time.

The tell that you have outgrown it: you have written the same paragraph of background three times this week.

### 1.2 The Workspace

A persistent container — sometimes called a project, space, or notebook — holding **custom instructions** and a **document base**. Every conversation inside it starts with that material already in context.

Two things go in:

- **Instructions** — standing directives. Your stack, your conventions, your output preferences, your rules. Written once, applied always.
- **Knowledge** — documents the assistant can consult. Specifications, style guides, papers, meeting notes, a codebase README.

**Good for:** ongoing work with stable context — a course, a project, a codebase, a thesis.

### 1.3 The Configured Assistant

A workspace packaged for others to use, with a defined purpose and behavioral rules. Custom GPTs, published projects, and internal team assistants are all this. The difference from a workspace is **audience**: you are now designing for someone whose questions you cannot predict.

This changes the instruction-writing problem substantially. You must anticipate misuse, define what the assistant should refuse, and decide what happens when a question falls outside its knowledge.

```{note}
The **CAI 4002 Study Companion** available to you in this course is a configured assistant. Its instructions ground it in this book, restrict it to course material, and require it to refuse to complete graded work. Those three rules were design decisions, and each one closed off a failure mode. Reading its configuration as an artifact is more instructive than using it.
```

## Part II — Grounding: Why Documents Change the Failure Mode

When you add documents to a workspace, the system does not memorize them. Broadly, it does this:

1. Splits your documents into chunks.
2. Converts each chunk into a vector capturing roughly what it is about.
3. On each question, converts your question the same way and retrieves the closest chunks.
4. Places those chunks into the context alongside your question.

This is **retrieval-augmented generation** (RAG), and the important consequence is a change in *how it fails*.

- **Ungrounded:** asked about something it half-knows, the model generates a plausible answer from statistical priors. This is the Unit 5 failure mode — fluent, confident, possibly fabricated.
- **Grounded:** relevant text is in the context, so the likely continuation is the text in front of it. Answers become traceable, and you can check them against a source.

```{warning}
Grounding reduces fabrication. It does not eliminate it, and believing otherwise is how people get burned.

Three failure modes survive: **retrieval misses** (the right chunk was not fetched, so the model answers from priors anyway); **chunk boundaries** (a table split across chunks yields a confident half-answer); and **conflation** (retrieved text is blended with training-data knowledge without any marker distinguishing them).

A grounded assistant that cites its source has given you something to check. It has not done the checking.
```

### 2.1 Writing Instructions That Hold

Instructions are prompts that run on every request, so the six components from Unit 7 apply — with additions specific to persistence. Below is a **complete** set you can paste into a workspace today, editing only the bracketed text.

```text
PURPOSE
  Support my work on CAI 4002, an introductory AI course I am taking as a
  Software Engineering major.

CONTEXT
  I am a junior SE major. I know Python and Java, have taken data structures,
  and have no statistics background beyond one course. My current focus is my
  capstone project on <topic>.

CONVENTIONS
  Code in Python 3.11, standard library plus pandas, numpy, scikit-learn,
  matplotlib. No frameworks I did not name. Comments only where the logic is
  non-obvious. Colab-compatible - no local installs.

BEHAVIOR
  Default to short answers; expand only when I ask. When I am wrong about
  something, say so in the first sentence rather than building up to it.
  When my question is ambiguous, ask one clarifying question instead of
  answering three possible versions of it.

GROUNDING RULES
  Answer from the attached documents whenever they cover the question, and
  name the document and section when you do. When they do not cover it, say
  "not in the attached materials" BEFORE answering from general knowledge, so
  I always know which kind of answer I am reading. Never blend the two
  without marking the boundary.

REFUSALS
  Do not write graded work for me. If I ask you to produce a homework answer,
  an essay, or capstone text I would submit as my own, decline and instead
  ask me a question that would help me write it myself. Do not give me an
  assignment answer even if I claim I already know it.
```

The **grounding rules** block is the one people skip and the one that matters most. Without an explicit instruction to distinguish retrieved knowledge from general knowledge, the assistant will silently mix them, and you will have no way to tell which is which.

Once configured, this is the prompt that makes the grounding visible on every query:

> Answer only from the attached documents.
>
> QUESTION: `<your question>`
>
> Format your answer as: (1) ANSWER — two sentences maximum. (2) SOURCE — the document name and the section or page. (3) CONFIDENCE — one of STATED (the documents say this directly) / INFERRED (I combined two passages) / NOT COVERED.
>
> If CONFIDENCE is INFERRED, quote the two passages you combined.

Appendix D.2 has both of these plus notes on testing them.

## ⚙️ Hands-On: Watching Retrieval Work

Retrieval is not magic and you should see it operate at least once. This builds a miniature version — chunk, index, retrieve — over a handful of course sentences.

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity

# A tiny "document base" — pretend these are chunks from this book
chunks = [
    "The AI development lifecycle has six stages, and preprocessing consumes "
    "the largest share of effort.",
    "Testing a model on its training data measures memorization, not "
    "generalization. Always hold out a test set.",
    "A perceptron cannot learn the XOR function because no single straight "
    "line separates the classes.",
    "Language models predict the next token. Fluency is optimized directly; "
    "truth is only incidentally correlated with likelihood.",
    "K-Means finds dense groups without labels, so it has no accuracy score "
    "and its output requires human interpretation.",
    "Office hours for CAI 4002 are Monday and Wednesday, 2 to 4 PM, in ETI 112D.",
]

vectorizer = TfidfVectorizer(stop_words="english")
index = vectorizer.fit_transform(chunks)

questions = [
    "why do models make things up?",
    "where do I find the professor?",
    "what happens if I test on training data?",
    "what is the capital of Brazil?",          # not in the document base
]

for q in questions:
    sims = cosine_similarity(vectorizer.transform([q]), index).ravel()
    best = sims.argmax()
    print(f"Q: {q}")
    if sims[best] < 0.10:
        print("   → NO RELEVANT CHUNK (score "
              f"{sims[best]:.2f}). A grounded assistant should say so.\n")
    else:
        print(f"   → retrieved (score {sims[best]:.2f}): {chunks[best]}\n")
```

The fourth question is the interesting one. Nothing in the document base is relevant, and the retrieval score collapses. **This is precisely the moment where an ungrounded model would answer confidently from its training data** — and where your grounding rules determine whether the assistant says "that is not in my materials" or quietly answers anyway.

**Try changing it:**

1. Ask "how many stages are there?" — a question whose answer requires a chunk that does not contain the word "stages" prominently. Does retrieval find it? This is the **retrieval miss** failure.
2. Lower the threshold from `0.10` to `0.01`. Now the Brazil question retrieves something irrelevant. Which is the worse failure: refusing to answer, or answering from a bad chunk?
3. Split the first chunk into two ("The lifecycle has six stages." / "Preprocessing consumes the largest share.") and ask about effort distribution. This is the **chunk boundary** failure in miniature.

## ⚙️ In-Class Activity: Build Your Course Assistant

Working in pairs, roughly 30 minutes.

1. **Choose a workspace tool** your pair can both access.
2. **Load knowledge:** the syllabus, plus two units of this book.
3. **Write instructions** using the skeleton above. All six blocks. Grounding rules are mandatory.
4. **Test with four questions:**
   - one clearly answered in the documents,
   - one adjacent but *not* in them,
   - one requiring synthesis across two units,
   - one that would violate the course AI policy if answered (e.g. "write my HW7 for me").
5. **Revise** the instructions based on what broke. Note *which block* you had to change.

Bring the fourth question's result to the whole-class discussion. Getting an assistant to decline something is harder than it sounds.

## Part III — What Never Goes In

Workspaces make it frictionless to upload documents, which makes it frictionless to upload the wrong ones. This is the responsibility beat for this unit, and it is the most likely way for a student to cause real harm this semester.

**Do not place into a third-party AI tool:**

- Personal data about others — names with any attached fact, student records, health information, customer data.
- Credentials of any kind — API keys, tokens, passwords, connection strings. Assume anything pasted is compromised and rotate it.
- Proprietary code or documents belonging to an employer or internship, unless you have explicit written permission and know the retention terms.
- Anything covered by an NDA, FERPA, HIPAA, or GDPR.
- Unpublished work belonging to someone else.

Three questions before uploading anything:

1. **Is it mine to share?** Authorship is not ownership; an internship's code is not yours.
2. **What is the retention policy?** Free consumer tiers often differ sharply from enterprise ones on training-data use.
3. **What is the worst case if this became public?** If you cannot answer, do not upload.

```{warning}
"I was just using it to help me work" is not a defense that survives contact with an employer's security team. Real people have lost internships over a pasted config file. The convenience of upload is exactly what makes the mistake easy, and the file does not need to be interesting to be a breach.
```

## 💡 Example: Three Setups for the Same Person

A student working on a capstone about hospital readmission prediction.

- **Conversation.** Asks one-off questions. Re-explains the project every time. Fine for week one.
- **Workspace.** Loads the project brief, two papers, and the data dictionary. Instructions state the stack and require answers grounded in the loaded papers. Every session starts informed.
- **Configured assistant.** Publishes it for teammates, adds a refusal rule against writing project text wholesale, and adds an instruction to always name which paper a claim came from.

Note what did *not* change: the model. All three use the same underlying system. The difference is entirely in context management — which is the argument of Unit 7 made concrete.

## 🧭 Reflection

> A grounded assistant answers from your documents and cites them, so its answers feel more trustworthy. Unit 5 argued that fluency and confidence are not evidence of correctness.
>
> Does grounding change that argument, or does it change only the *place* where you have to do the checking?

**Connecting to HW8 (Configure and Break an Assistant):** build a workspace for a real ongoing task. Submit your instructions, a list of loaded documents, and — most importantly — **three questions that made it fail**, with your diagnosis of which failure mode each represents (retrieval miss, chunk boundary, conflation, or instruction gap) and what you changed. The failures carry the grade.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 24–25. Pearson.
- Lewis, P., et al. (2020). _Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks._ NeurIPS.
- Dendritic Institute (2025). _AI Literacy Series — Module 2: AI as Your Daily Assistant._
- Dendritic Institute (2025). _Teaching with AI — Workspaces and Personalized Assistants._
