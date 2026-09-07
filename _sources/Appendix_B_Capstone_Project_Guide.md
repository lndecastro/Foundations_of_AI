# Appendix B: Capstone Project Guide

> **Guidelines released:** Session 21 (Oct 28) · **Presentations:** Wednesday, Dec 02, 2026, 10:00 AM – 12:15 PM, Holmes Engineering 402

In place of a traditional final exam, you will complete a capstone project and deliver a formal presentation. This is the culminating demonstration of everything the course covered.

## The Assignment

Identify a **real-world problem or opportunity** in a domain of your choosing — or accept a challenge assigned by the instructor — and propose an **AI-based solution**. Your project must draw on course concepts including AI fundamentals, relevant applications, data considerations, and ethical implications.

### Format

| Requirement | Detail |
| :--- | :--- |
| **Individual or pairs** | Pairs require instructor approval |
| **Length** | 10–12 minutes, followed by brief Q&A |
| **Materials** | Slide deck or visual aid required |
| **Evaluation** | Clarity, depth of analysis, use of course concepts, solution quality |

### Grading Criteria

| Criterion | Weight |
| :--- | :---: |
| Problem definition and relevance | 20% |
| Understanding and application of AI concepts | 30% |
| Proposed solution and feasibility | 35% |
| Presentation clarity and delivery | 15% |

```{important}
**Feasibility carries the most weight (35%).** This rewards a modest proposal you have thought through completely over an ambitious one you have not. A well-scoped system that could plausibly be built next year will outscore a visionary platform that hand-waves past its data requirements.
```

## A Structure That Maps to the Rubric

### 1. Problem Definition (→ 20%)

- What is the problem, and **who has it**? Be specific about the affected people.
- How is it handled today, and what does that cost?
- What would improvement actually look like — and how would you know you achieved it?
- **Why is this an AI problem** rather than a process, staffing, or policy problem?

> That last question is the one most projects skip and the one that separates strong proposals from weak ones. Many real problems are better solved without AI. Saying so where it is true demonstrates judgment, not weakness.

### 2. AI Concepts (→ 30%)

Use the vocabulary of this course precisely:

- **Task type** (Unit 2) — classification, regression, clustering, generation, ranking, sequential decision?
- **Learning paradigm** (Unit 4) — supervised, unsupervised, reinforcement? Why that one?
- **Approach** (Units 3, 5) — what family of model, and what makes it appropriate?
- **Evaluation** (Units 3, 10) — which metric, and *why that metric*? What does it hide?

```{note}
Naming your metric and defending the choice is one of the highest-value paragraphs in the whole presentation. "We'd measure accuracy" is weak. "We'd prioritize recall, because a missed case costs far more than a false alarm here" demonstrates that you understood Unit 3.
```

### 3. Solution and Feasibility (→ 35%)

- **Data**: what exactly would you need? Who holds it? Does it exist in usable form? What would it cost to obtain?
- **Data quality** (Unit 9): completeness, representativeness, provenance. Run the **leakage check** on every proposed feature.
- **Failure modes** (Unit 5): what happens when it is wrong? What happens when the world shifts?
- **Fairness** (Unit 10): which groups are affected, which fairness definition applies, how would you audit?
- **Human role** (Unit 11): advisory or automated? Who reviews, who overrides, who is accountable?
- **Governance** (Unit 12): which risk tier? What obligations follow?

### 4. Delivery (→ 15%)

- Rehearse with a timer. **10–12 minutes is short.**
- Assume an intelligent audience without technical background — the same audience this course was designed for.
- Prepare for the obvious questions: *where does the data come from?*, *what if it's wrong?*, *why not just do this manually?*

## Strengthening Your Project

Two deliverables from earlier units transfer directly:

**A model card (Unit 11).** Fill out the template for your proposed system. Whichever field is hardest to complete honestly is pointing at your weakest area — fix that before the presentation.

**A risk classification (Unit 12).** Run your system through the risk-tier classifier. If it lands in **high risk**, address the obligations that follow. This is exactly the analysis a real proposal would require.

## Common Failure Modes

| Pitfall | What it looks like | Fix |
| :--- | :--- | :--- |
| **Solution in search of a problem** | "I want to use an LLM for..." | Start from the problem |
| **Data hand-waving** | "We would collect user data" | Name the source, format, volume, owner |
| **Ethics as an afterthought** | One slide at the end | Weave through the analysis |
| **Ignoring failure** | Only the success case shown | Show what breaks and who it hurts |
| **Overscoping** | A platform solving six problems | Solve one thing well |
| **Unexamined metrics** | "We'd get 95% accuracy" | Say which metric and why (Unit 3) |

## Suggested Timeline

| When | Milestone |
| :--- | :--- |
| **Oct 28 (Session 21)** | Guidelines released; begin problem selection |
| **Early November** | Problem chosen; confirm pairs with instructor |
| **Nov 02–04 (Unit 9)** | Complete data feasibility analysis |
| **Nov 09–16 (Unit 10)** | Complete fairness analysis |
| **Nov 18–23 (Units 11–12)** | Model card and risk classification |
| **Late November** | Build slides; rehearse with a timer |
| **Dec 02** | Present |

## AI Use on the Capstone

Per the syllabus, AI use is **permitted with disclosure**. AI may assist with **research and drafting only** — your final analysis, arguments, and conclusions must reflect your own thinking.

Include this at the end of your submission:

> **AI Disclosure:** I used [Tool Name] to [brief description, e.g. brainstorm ideas, summarize a source, check grammar]. The final responses and conclusions are my own.

See **Appendix C** for full details on disclosure requirements.

```{warning}
An AI can produce a plausible-sounding capstone in minutes. It cannot answer the Q&A. The questions after your presentation are where genuine understanding becomes visible, and they are worth preparing for more carefully than the slides.
```
