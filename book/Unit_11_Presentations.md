# Unit 11: Presentations

> **Session 17** · Oct 14 · **Part of HW10 (Office Portfolio), due Oct 21**

Six weeks from now you will each deliver a capstone presentation. This session is therefore not incidental — it is direct preparation for a graded deliverable, and you should treat the techniques here as tools you will use in November.

The engineering context matters. Software engineers present constantly: architecture reviews, sprint demos, incident briefings, design proposals, and conference talks. Most engineering presentations fail the same way, and it is not a design failure. It is a **translation failure** — the presenter knows the material so well that they cannot see what the audience does not know.

AI tools are surprisingly good at that specific problem, and surprisingly bad at almost everything else about presentations.

## Learning Objectives

After completing this unit, you will be able to:

- Separate the parts of presentation work that AI helps with from the parts it damages.
- Generate a slide deck from a structured outline and evaluate the result critically.
- Apply the **one idea per slide** and **assertion-evidence** principles.
- Rehearse against AI-generated audience questions.

## Part I — Where AI Helps and Where It Hurts

| Genuinely useful | Do it yourself |
| :--- | :--- |
| Translating technical depth for a non-technical audience | Deciding what the presentation argues |
| Generating a first structural outline | Choosing what to cut |
| Turning a design doc into slide-sized units | The actual technical claims |
| Producing speaker notes from your bullets | Anything with a number on it |
| Anticipating audience questions | The delivery |

```{warning}
AI presentation generators produce **plausible decks**, and plausible is the failure mode. Ask for "a presentation about my capstone project" and you will get twelve slides in a familiar shape — Introduction, Background, Methodology, Results, Conclusion — with generic content in each. It will look finished. It will argue nothing.

A presentation with no argument is worse than no presentation, because it consumes the audience's time and yours without transferring anything.
```

## Part II — Structure Before Slides

The order matters and most people get it backwards. **Do not open a slide tool first.**

### 2.1 The One-Sentence Test

Before anything else, write the single sentence you want the audience to repeat afterwards. Not a topic — a claim.

- Topic: *"My capstone on hospital readmission."* (Nobody repeats this.)
- Claim: *"Readmission models trained on billing data learn coding practice, not clinical risk."* (This is worth thirty seconds of someone else's attention.)

If you cannot write the claim, you are not ready for slides, and no tool will rescue you.

### 2.2 Assertion-Evidence

The dominant convention in technical presentations — a topic phrase headline over a bulleted list — is a poor format, because the headline carries no information and the bullets carry too much.

The alternative:

- **Headline is a full sentence stating the claim.** "Accuracy hides the error that matters" rather than "Model Evaluation."
- **Body is visual evidence for that claim.** A chart, a diagram, a code fragment, an image.
- **Speaker supplies the connective reasoning.** Not the slide.

This is a better format for one specific reason: a reader skimming your headlines alone should get the whole argument. Try that with topic headlines and you get a table of contents.

### 2.3 One Idea Per Slide

If a slide needs two ideas, it needs to be two slides. Slides are free; audience attention is not.

## ⚙️ Hands-On: Outline to Deck

The reliable workflow is **you write the argument, the tool writes the slides.** Never the reverse.

```python
# Step 1: write your argument as a structured outline BEFORE touching any tool.
# Each headline is a full-sentence claim. Each evidence line is what will
# occupy the slide body.

outline = {
    "claim": ("Readmission models trained on billing data learn coding "
              "practice, not clinical risk."),
    "audience": "Engineering faculty and classmates; technical, not clinical.",
    "duration_min": 11,
    "slides": [
        ("Hospitals lose money on readmissions, so everyone wants to predict them.",
         "One chart: readmission penalty cost, national"),
        ("The obvious data source is the billing record.",
         "Diagram: what a billing record contains"),
        ("A model on billing data reaches 91% accuracy.",
         "Confusion matrix"),
        ("That number is measuring the wrong thing.",
         "Highlight: the discharge-code column, filled in after the outcome"),
        ("Removing the leaked column drops accuracy to 68%.",
         "Same confusion matrix, side by side"),
        ("68% is the honest number, and it is still useful.",
         "Cost curve: value at 68% vs. clinician baseline"),
        ("Any model using post-outcome fields should be assumed leaked.",
         "Checklist: four questions to ask of any feature"),
    ],
}

print(f"CLAIM: {outline['claim']}")
print(f"AUDIENCE: {outline['audience']}")
print(f"BUDGET: {outline['duration_min']} min "
      f"→ {outline['duration_min'] * 60 / len(outline['slides']):.0f}s per slide\n")

print("HEADLINE-ONLY READ-THROUGH (does the argument survive?)")
print("-" * 68)
for i, (headline, _) in enumerate(outline["slides"], 1):
    print(f"{i}. {headline}")

print("\nSLIDE PLAN")
print("-" * 68)
for i, (headline, evidence) in enumerate(outline["slides"], 1):
    print(f"[{i}] {headline}\n     evidence: {evidence}")

# Step 2: paste the SLIDE PLAN into a presentation tool as the generation prompt.
# Step 3: fix everything it invents.
```

The **headline-only read-through** is the diagnostic. Read those seven sentences with nothing else. If they form an argument, the deck will work. If they read as a list of topics, no amount of visual design will save it.

**Try changing it:**

1. Do the read-through on your own capstone outline. Does it argue, or does it list?
2. Cut two slides. Which two? The exercise of finding them tells you which slides were carrying the argument.
3. Change `audience` to "hospital administrators, non-technical" and rewrite three headlines. What changed — the claims, or the evidence?

## ⚙️ In-Class Activity: Generate, Then Repair

Working in pairs, roughly 30 minutes.

1. **Write an outline** in the form above for a topic from Units 4–9. Ten minutes, no tools.
2. **Generate a deck** by pasting the slide plan into an AI presentation tool.
3. **Audit the output against this checklist:**
   - Which headlines were changed from claims into topics? (This is the most common regression.)
   - What content appeared that you did not supply? Every instance is fabricated.
   - Are there any numbers on the slides? Where did they come from?
   - How many ideas are on the busiest slide?
4. **Repair it.** Restore your headlines, delete invented content, fix the busiest slide.
5. **Report** to the class: what did the tool do well, and what did it silently change?

```{note}
Step 3's second question is the one that matters. AI presentation tools fill empty space, because a sparse slide looks unfinished to a system optimizing for typical output. The filling is generated, not retrieved, and it will be confidently wrong about your project in ways only you can detect.
```

## Part III — Rehearsal and Questions

The most undervalued use of AI in presentations is not building slides. It is **adversarial rehearsal**.

> *You are a skeptical engineering professor attending this 11-minute presentation. Here is my outline. Generate the eight hardest questions you would ask, ordered by how much they would damage my argument if I could not answer them. For each, note what a weak answer would look like.*

This works well because the questions come from statistical regularity — the model produces the questions *usually* asked of arguments shaped like yours. That is precisely the set you should be prepared for.

Then, separately:

> *Here is my answer to question 3. Where is it weak?*

```{warning}
Do not ask it to write your answers. In the capstone Q&A you will be answering live, and a memorized answer collapses under one follow-up. Use it to find the questions; find the answers yourself, in the material.
```

## 💡 Example: The Same Content, Two Decks

**Deck A** — generated from "make a presentation about bias in machine learning." Twelve slides. Headlines: *Introduction, What is Bias, Types of Bias, Real-World Examples, Mitigation Strategies, Conclusion.* Every slide has four bullets. It is polished, it is complete, and it could have been generated for any student in any course in any year.

**Deck B** — generated from a seven-line outline written by a student who had spent two weeks on one hiring dataset. Headlines are claims. Slide four is a single confusion matrix with one cell circled.

Deck B is worse-looking and better. The difference is not the tool — both used the same one. The difference is that Deck B's author knew what they were arguing before they generated anything.

## 🧭 Reflection

> A tool can now produce a competent-looking deck from a one-line request in about ninety seconds.
>
> If the deck looks finished and argues nothing, has the tool helped you or has it helped you avoid noticing that you had nothing to say yet?

**Connecting to HW10 (Office Portfolio) and the Capstone:** for Unit 11, submit a **five-to-seven slide outline in the format above** for your capstone, plus the headline-only read-through, plus the eight adversarial questions and your own answers to the three hardest. This is graded as Unit 11 work and is also the scaffold for your November presentation — the work is not duplicated.

## 📘 Further Reading

- Alley, M. (2013). _The Craft of Scientific Presentations_, 2nd Ed. Springer. (Origin of the assertion-evidence structure.)
- Tufte, E. (2006). _The Cognitive Style of PowerPoint_, 2nd Ed. Graphics Press.
- Dendritic Institute (2025). _AI Literacy Series — Module 3, Part III: AI for Presentations._
