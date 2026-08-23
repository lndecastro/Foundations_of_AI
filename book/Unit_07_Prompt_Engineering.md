# Unit 7: Prompt Engineering

```text
Sessions 11–12 | Sep 23, 28 | HW7 assigned Sep 28, due Oct 05
```

Part III begins here. For six units you have been taking these systems apart; from now on you are operating them. Everything you build in Units 8 through 15 rests on the skill in this unit.

Prompting has a bad reputation among engineers, and some of it is deserved — the genre of "10 magic prompts" content is worthless. But the underlying skill is real and it is not mysterious. **A prompt is a specification.** You already know how to write specifications; you know that vague requirements produce wrong software, and that the fix is precision about inputs, outputs, constraints, and acceptance criteria. Prompting is the same discipline applied to a nondeterministic executor.

## Learning Objectives

After completing this unit, you will be able to:

- Decompose a prompt into its functional components and write each deliberately.
- Apply **role, few-shot, chain-of-thought, and output-format** techniques and say when each helps.
- Distinguish **prompt engineering** from **context engineering** and explain why the second matters more at scale.
- Design a **verification step** appropriate to the task's error cost.
- Build and maintain a reusable **prompt library**.

## Part I — Why Prompting Is Not Conversation

The failure mode for new users is treating the model as a colleague who shares your context. It does not. It has no access to your repository, your team's conventions, last week's meeting, or what you actually meant. It has the text in front of it and a very large statistical prior over what usually follows such text.

### 1.1 Anatomy of an Effective Prompt

Six components. Not all are needed every time, but knowing which you omitted is the point.

| Component | Question it answers | Example |
| :--- | :--- | :--- |
| **Role** | From what perspective? | "You are a senior backend engineer reviewing a pull request." |
| **Task** | What action, exactly? | "Identify race conditions in this function." |
| **Context** | What must it know? | "This runs in a multi-threaded request handler. `cache` is shared." |
| **Constraints** | What are the limits? | "Python 3.11, standard library only, no external dependencies." |
| **Format** | What shape is the output? | "A markdown table: line number, issue, severity, suggested fix." |
| **Criteria** | What makes it good? | "Flag only issues that could produce incorrect data, not style." |

```{note}
The component that beginners omit most often is **Criteria**, and it is the one that changes the output most. Without it, the model optimizes for what usually satisfies such a request — which is *thoroughness*. That is why unconstrained prompts return twelve suggestions when you wanted the two that matter.
```

### 1.2 Core Techniques

**Role prompting.** Assigning a perspective narrows the statistical territory the model draws from. "Explain OAuth" and "Explain OAuth to a backend developer who has implemented session auth but never a token flow" produce genuinely different text, because the second describes a much narrower region of the training distribution.

**Few-shot prompting.** Give two or three worked examples of input → output. This is by far the most reliable technique for enforcing a format, and it usually beats describing the format in words. If you need commit messages in a house style, showing three real ones works better than a paragraph of rules.

**Chain-of-thought.** Asking the model to work through steps before answering improves multi-step reasoning. The mechanism is worth understanding: the intermediate tokens become part of the context for the tokens that follow, so the model is conditioning on its own partial work. It is not "thinking harder" — it is giving itself more relevant context.

**Output format specification.** Ask for JSON, a table, or a fixed template when the output will be consumed by anything other than a human reading prose. This is the technique that makes AI output composable with the rest of your tooling.

**Negative constraints.** "Do not include explanations." "If the answer is not in the provided text, say so." The second is one of the highest-value instructions you can write, for reasons Unit 5 made clear.

### 1.3 Iteration Is the Method

The first prompt is a draft. Professionals do not write one good prompt; they run a short loop:

1. Write the prompt with all six components.
2. Run it. Read the output **against your criteria**, not for general quality.
3. Identify the single largest gap.
4. Change one thing. Run again.

Changing one thing at a time is not fussiness. If you change four things and the output improves, you have learned nothing transferable.

## ⚙️ Hands-On 1: Comparing Prompts Systematically

This cell prints four variants of the same request, from bare to fully specified. Paste each into an AI assistant **in a separate conversation** — a fresh one each time, so earlier context does not leak — and score the results.

```python
task = "Explain database indexing."

variants = {
    "V1 — bare": task,

    "V2 — + role and audience":
        "You are a database engineer explaining to a junior developer who "
        "knows SQL but has never tuned a query. " + task,

    "V3 — + context and constraints":
        "You are a database engineer explaining to a junior developer who "
        "knows SQL but has never tuned a query. " + task + " "
        "Focus on B-tree indexes in PostgreSQL. Assume tables of ~10 million "
        "rows. Do not discuss full-text or geospatial indexes.",

    "V4 — + format and criteria":
        "You are a database engineer explaining to a junior developer who "
        "knows SQL but has never tuned a query. " + task + " "
        "Focus on B-tree indexes in PostgreSQL. Assume tables of ~10 million "
        "rows. Do not discuss full-text or geospatial indexes.\n\n"
        "Format: (1) a two-sentence intuition, (2) one concrete SELECT that "
        "is slow without an index, (3) the CREATE INDEX that fixes it, "
        "(4) one case where adding an index makes things worse.\n"
        "Criteria: a reader should be able to decide whether to add an index "
        "to their own table after reading this. Under 300 words.",
}

for name, prompt in variants.items():
    print("=" * 72)
    print(name, f"({len(prompt.split())} words)")
    print("-" * 72)
    print(prompt, "\n")

print("=" * 72)
print("""SCORING RUBRIC — rate each output 1-5, then total

  Accuracy      Is everything stated actually true?
  Relevance     Does it address the audience described?
  Actionability Could the reader do something differently after reading?
  Concision     Is anything present that earns no place?
""")
```

**Try changing it:**

1. Score all four. Where is the largest jump — V1→V2, V2→V3, or V3→V4? The answer varies by task, which is exactly the point.
2. Write a V5 that adds one few-shot example. Does it beat V4 on **Concision**?
3. Replace the task with something from your own coursework and rebuild all four variants. Which component did you find hardest to write? That is your weak spot.

## Part II — Context Engineering

Prompt engineering is what you write in the box. **Context engineering** is the management of everything the model can see: system instructions, retrieved documents, conversation history, tool outputs, and files.

This distinction matters because in real deployments the prompt is a small fraction of the context, and the failures come from the rest of it.

- **Context is finite.** Every model has a limit. Exceeding it means something gets dropped, and you rarely control what.
- **Position matters.** Material at the very beginning and very end of a long context is attended to more reliably than material buried in the middle. This is a measured, reproducible effect, not folklore.
- **Irrelevant context actively harms.** Padding a prompt with loosely related documents makes output worse, not better. More is not more.
- **Stale context persists.** In a long conversation, a correction you made twenty turns ago competes with the original error, which is still sitting there in the context.

```{warning}
The practical consequence: **start a fresh conversation more often than feels necessary.** When a thread has gone badly wrong, patching it with "no, I meant..." is usually worse than restating the task cleanly in a new one. Half of what looks like model stubbornness is context contamination.
```

Unit 8 takes this further — workspaces and assistants are, mechanically, tools for controlling context deliberately instead of accidentally.

## Part III — Verification Is Part of the Prompt

Unit 5 established the mechanism: fluency is optimized, truth is incidental, and confidence is uncorrelated with correctness. That fact has an operational consequence, and this is where the course's AI policy actually comes from.

**Match your verification effort to the error cost.** Not everything needs the same scrutiny.

| Error cost | Example | Verification |
| :--- | :--- | :--- |
| **Trivial** | Brainstorming names, rephrasing a sentence | Read it. Done. |
| **Low** | Draft email, first outline | Skim for anything factual; check those. |
| **Moderate** | Code you will run, a summary you will forward | Execute it, or check against the source. |
| **High** | Anything with a citation, number, API, or legal claim | Verify every one, independently. |
| **Severe** | Anything affecting a person's money, health, safety, or record | Do not delegate the judgment at all. |

Three specific habits worth building now:

- **Citations are guilty until proven innocent.** Fabricated references are the single most common serious failure. They look right — plausible authors, plausible journals, plausible years. Open them.
- **Numbers are guilty until proven innocent.** A model asked to compute will often produce a well-formatted answer with no arithmetic behind it. Unit 12 makes you catch this in a spreadsheet.
- **Ask it to mark its own uncertainty, then distrust the marking.** "Flag anything you are not confident about" surfaces some errors and misses others, because the model has no reliable access to its own uncertainty. It is a filter, not a guarantee.

## ⚙️ Hands-On 2: Building Your Prompt Library

A prompt library is a set of prompts you have refined and can reuse. Professionals keep one. This cell ships with **five complete entries** — not fragments, but the full text you would actually send. Run it, read them, then start replacing them with your own.

```python
library = {

"code_review": {
"prompt": """You are a senior engineer reviewing a pull request.

Identify correctness bugs in the code below.

Ignore formatting, naming, and style. Flag only issues that could produce
incorrect behavior at runtime: race conditions, off-by-one errors, unhandled
None, incorrect boundary conditions, resource leaks.

Format as a table: line | issue | why it breaks | minimal fix.

If there are no correctness bugs, say "No correctness bugs found" and stop.
Do not pad the table to seem thorough.

CODE:
<paste code here>""",
"verify": "Reproduce each claimed bug with a test before believing it. "
          "A plausible bug report is not a bug report."},

"explain_unfamiliar_code": {
"prompt": """You are helping a new teammate onboard to a codebase you know well.

Explain what the code below does and why it exists.

The reader knows the language but has never seen this codebase. Do not explain
language features. Do explain anything specific to this system.

Format: one paragraph of purpose, then a bulleted walkthrough of the logic.

Name every assumption the code makes about its inputs - encoding, ordering,
nullability, size, type. Mark anything you inferred from a name rather than
from the logic as [INFERRED].

CODE:
<paste code here>""",
"verify": "Check the walkthrough against the code line by line. "
          "Verify every [INFERRED] claim separately."},

"debug_with_context": {
"prompt": """I have a bug I cannot locate. Do not guess at the cause.

First, list the five most likely causes given the symptoms, ordered by
probability, and for each one state the single cheapest test that would rule
it in or out.

Do not propose a fix until I tell you which test result I got.

SYMPTOM: <what you observe>
EXPECTED: <what should happen>
WHAT I HAVE ALREADY RULED OUT: <list, so it does not repeat your work>
RELEVANT CODE: <paste>
ERROR OUTPUT: <paste verbatim, do not summarize>""",
"verify": "Run the tests in the order given. Stop at the first that "
          "changes your picture; do not run all five."},

"summarize_a_thread": {
"prompt": """Below is a long discussion thread. I need to act on it.

Extract, in this order:
1. DECISIONS MADE - only ones explicitly agreed, with who agreed.
2. OPEN QUESTIONS - things raised and not resolved.
3. ACTION ITEMS - with owner and deadline where stated, [NO OWNER] where not.
4. DISAGREEMENTS - positions still in conflict, stated fairly for both sides.

Do not summarize the discussion. Do not infer agreement from the absence of
objection. If someone proposed something and nobody responded, that belongs
under OPEN QUESTIONS, not DECISIONS MADE.

THREAD:
<paste>""",
"verify": "Check every DECISION against the thread - the most common error "
          "is promoting a proposal nobody objected to into a decision."},

"stress_test_my_reasoning": {
"prompt": """I have reached a conclusion and I want it attacked, not confirmed.

MY CONCLUSION: <state it plainly>
MY REASONING: <how you got there>
EVIDENCE I AM RELYING ON: <list>

Give me the three strongest objections. For each: what specifically is wrong
or unsupported, and what evidence would settle it.

Do not begin by telling me the reasoning is sound. Do not soften. If the
conclusion survives all three objections, say so at the end - but find the
three first.""",
"verify": "Take the strongest objection and actually check it. An objection "
          "you read and dismissed is not a check."},

# ADD YOUR OWN - at least five, for tasks you genuinely repeat.
}

for name, entry in library.items():
    print("=" * 72)
    print(f"[{name}]\n")
    print(entry["prompt"])
    print(f"\n  -> VERIFY: {entry['verify']}\n")

print("=" * 72)
print(f"{len(library)} entries. Appendix D has ~40 more, organized by unit.")
```

Note that every entry carries a `verify` field, and note what the prompts have in common: each one **forbids the reassuring answer**. *Do not pad the table. Do not infer agreement from absence of objection. Do not begin by telling me the reasoning is sound.* The statistically typical continuation is the smooth, complete-looking, agreeable one — so constraining the output away from it is most of the technique.

**Appendix D** contains the full course library — roughly forty complete prompts organized by unit, plus universal add-ons you can append to any of them. Use it as the model for your own entries.

**Try changing it:**

1. Add three entries for tasks you genuinely repeat. Vague ones ("write better") do not count.
2. For one entry, deliberately omit `criteria` and compare outputs. Was the difference what you predicted?
3. Add a `failure_modes` field recording what went wrong when you used it. After a month this field is the most valuable part of the library.

## 💡 Example: Weak Prompt vs. Strong Prompt

**Weak.** *"Write tests for my function."*

The model does not know the language, the test framework, what the function does, what edge cases matter, or whether you want unit or integration tests. It will guess all six, plausibly, and you will get pytest when your repository uses unittest.

**Strong.** *"You are writing unit tests for a Python 3.11 codebase that uses pytest. Below is a function that parses ISO 8601 duration strings into `timedelta`. Write tests covering: valid input, malformed input, empty string, and the boundary where weeks and days are both present. Use `pytest.mark.parametrize`. Do not test the standard library's behavior — only this function's. Return only the test file, no commentary."*

Every added clause removes a guess. That is all prompt engineering is.

## 🧭 Reflection

> You just spent a session getting better at writing specifications for a system that cannot tell you when it has misunderstood you.
>
> Does a well-engineered prompt make the output more likely to be *correct*, or only more likely to be *what you asked for*? Are those the same thing, and does the difference matter for the verification habits you plan to adopt?

**Connecting to HW7 (Prompt Iteration Log):** take one real task and document five successive prompt revisions. For each, record what you changed, why, and what improved or degraded. Submit the log and the final prompt as a library entry — including its `verify` field. The log is graded, not the final prompt.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 24. Pearson.
- Mitchell, M. (2020). _Artificial Intelligence: A Guide for Thinking Humans_, Ch. 11–13. Penguin Books.
- Anthropic (2025). _Prompt Engineering Overview._ <https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview>
- Dendritic Institute (2025). _AI Literacy Series — Module 2, Part VII: Your Prompt Library._
