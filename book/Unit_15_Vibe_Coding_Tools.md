# Unit 15: Vibe Coding Tools

```text
Sessions 22–23 | Nov 02, 04 | Assessed as SE (Seminar): group tutorial, delivered in Session 23
```

This unit works differently from every other unit in the course. Session 22 is a shared foundation delivered by the instructor; **Session 23 is taught by you.** Each group investigates one tool and delivers a short tutorial to the class.

The format is deliberate. There are more of these tools than any one person can evaluate, they change every few months, and the durable skill is not knowing one of them — it is being able to assess an unfamiliar one quickly and honestly. You practice that by doing it.

## Learning Objectives

After completing this unit, you will be able to:

- Define what "vibe coding" describes and where it sits on a spectrum of AI coding assistance.
- Evaluate an AI coding tool against a structured rubric rather than an impression.
- Identify the license, security, and maintainability risks specific to generated code.
- Deliver a concise technical tutorial and defend your evaluation.

## Part I — What Vibe Coding Is

The term was coined in early 2025 to describe a way of working in which you describe what you want in natural language, accept what the system produces, and iterate by describing changes — without reading the generated code closely, or at all.

Note what is distinctive. This is not "AI helps me write code." It is **not reading the code** that defines the practice.

### 1.1 The Spectrum

| Level | What you do | What you read | Appropriate for |
| :--- | :--- | :--- | :--- |
| **Completion** | Type; accept suggestions | Every line | Production code |
| **Assisted** | Describe a function; review carefully | Every line | Production code |
| **Delegated** | Describe a feature; review the diff | The diff | Internal tools, with tests |
| **Vibe** | Describe an outcome; accept and iterate | Little or none | Prototypes, throwaways, exploration |

```{note}
Each level is legitimate **for its row**. The professional failure is not using vibe coding; it is using it for the wrong row and not noticing you have. Code written at the vibe level and shipped to production is not a stylistic choice — it is unreviewed code in a system that someone depends on.
```

### 1.2 Where It Genuinely Works

- **Prototypes to be thrown away.** If the artifact's purpose is answering "would this be worth building?", reading the code is wasted effort.
- **Exploratory analysis** you will rewrite once you know what you are looking for.
- **Personal tools** with one user and no consequences for failure.
- **Unfamiliar territory.** Getting something running in a stack you do not know, before learning it properly.

### 1.3 Where It Fails

- **Anything handling other people's data.** You cannot assess a security property of code you did not read.
- **Anything you will maintain.** Unread code becomes legacy code the day it is written, without ever having been understood.
- **Anything with correctness requirements you cannot test.** Tests are the only substitute for reading, and they only cover what you thought to test.
- **Anything with licensing exposure.** Discussed below.

## Part II — Three Risks Specific to Generated Code

This is the responsibility beat for Unit 15, and each risk is one your group must address in its tutorial.

### 2.1 Review Burden

Unit 6 named this: generation is fast, review is not, and review is the part that never scaled. A tool that produces 400 lines in ten seconds has not saved you time if reviewing 400 lines takes two hours — it has moved the work and made it less pleasant.

The honest question for any tool is not "how fast does it generate?" but **"what is the total time to a change I would defend in review?"**

### 2.2 License and Provenance

Generated code is produced by models trained on public repositories, including copyleft-licensed ones. When a tool emits a substantial block closely resembling GPL-licensed source, the legal position is unsettled and actively litigated.

Practical posture:

- Assume anything distinctive and non-trivial may have a provenance you cannot see.
- For anything commercial, check your organization's policy — most now have one.
- Treat a generated implementation of a *well-known algorithm* differently from a generated implementation of *something oddly specific*. The second is likelier to be a near-copy of something particular.

### 2.3 Security

Generated code reproduces the security properties of its training data, and public repositories are full of insecure patterns. The recurring ones: string-concatenated SQL, missing input validation, permissive CORS, secrets in source, outdated cryptographic choices, and dependencies that no longer exist or were never real.

That last one has a name — **slopsquatting** — where an attacker registers a package name that models frequently hallucinate, so the fabricated import resolves to hostile code. Every generated dependency should be verified to exist before it is installed.

```{warning}
The security risk compounds with vibe coding specifically. At the completion or assisted level, you read the SQL string and see the concatenation. At the vibe level, nobody does — by definition. The practice that maximizes speed is the practice that removes the review step which would have caught the vulnerability.
```

## ⚙️ Session 22 — Instructor-Led: The Shared Baseline

We build one small thing together at the delegated level, then examine what we got. The task is deliberately modest: a command-line tool that reads the sprint CSV from Unit 12 and reports completion rate by sprint with a threshold warning.

Working through it, we will note:

1. How many prompts were needed to reach something runnable.
2. What the generated code assumed that we never specified.
3. Every dependency it introduced, and whether each actually exists.
4. What a security review would flag.
5. Whether the total time beat writing it directly.

Item 2 is the one that repays attention. Generated code is full of unstated assumptions — encodings, date formats, error behavior, edge cases — and each one is a decision made on your behalf by a system optimizing for typicality.

## Part III — The Group Tutorial (Session 23)

### 3.1 Assignment

Groups of three or four. **Your tool will be assigned**, not chosen — part of the exercise is evaluating something you did not pick. Assignments are posted in Canvas on Oct 28.

```{warning}
**Verify access before you plan.** Several of these tools have credit caps or trial limits that expire mid-project. Confirm your whole group can use the assigned tool within the free tier during the first 48 hours, and tell me immediately if not. Do not pay for anything for this assignment.
```

### 3.2 Format

**Seven minutes per group. Strictly enforced** — three groups per session-half plus discussion, and the schedule has no slack.

Every tutorial covers, in this order:

1. **What it is** (1 min) — what the tool does, where it sits on the §1.1 spectrum, what it costs.
2. **Live demo** (2 min) — one task, start to finish. Pre-record as a fallback; live demos fail.
3. **Where it broke** (2 min) — the most valuable segment. What did you ask for that it got wrong? Show the actual output.
4. **The three risks** (1 min) — review burden, licensing posture, one security observation.
5. **Verdict** (1 min) — which row of the §1.1 table would you use this for, and which would you refuse?

### 3.3 The Common Task

So the tutorials are comparable, every group gives its assigned tool this exact prompt, unmodified, as its opening request:

```text
Build a single-page web application that:
1. Loads a CSV file chosen by the user with columns date, severity, minutes_to_resolve.
2. Displays the distribution of minutes_to_resolve broken down by severity.
3. Lets the user set a threshold, and flags any severity level whose median exceeds it.

Constraints: runs in a browser with no backend. No paid services. Handle a CSV with quoted commas, blank rows, and mixed date formats.
```

Record: prompts needed to reach something **runnable**, prompts needed to reach something **correct**, every assumption it made that you never specified, and what broke first.

Then run this against whatever it produced — it is the prompt that generates most of your §3.2 segment 3:

```text
Below is code generated by an AI tool. I did not write it and have not reviewed it. Tell me, in this order: (1) every assumption it makes about its inputs — encoding, format, ordering, nullability, size; (2) every external dependency it imports, and for each, whether that package actually exists on the public registry; (3) every place where untrusted input reaches a query, a file path, a shell command, or the DOM; (4) every error path that is silently swallowed. Do not tell me the code is good. Do not summarize what it does.
```

```{warning}
Check each named dependency on the registry **yourself**. An assistant asked whether a package exists will often say yes, because packages usually do exist — which is exactly the slopsquatting failure from §2.3, reproduced one level up.
```

Appendix D.9 has both of these plus a licensing-posture prompt.

### 3.4 Grading

| Criterion | Weight |
| :--- | :--- |
| Failure analysis — specific, reproduced, explained | **35%** |
| Risk assessment — licensing, security, maintenance | 25% |
| Demonstration quality and clarity | 20% |
| Verdict, defended against questions | 20% |

Note the weighting. **A tutorial that only shows the tool working scores poorly.** Every one of these tools has a demo that goes well; that is what the vendor's marketing is for. Your contribution is the part the marketing omits.

## 💡 Example: Two Evaluations of the Same Tool

**Group A** demonstrates a working app built in four minutes. Impressive. Verdict: "great tool, we'd use it for everything."

**Group B** demonstrates the same app, then shows that the CSV parser silently drops rows with quoted commas, that the threshold config was hard-coded despite being asked for as configurable, that one suggested dependency did not exist on the registry, and that the date parsing assumed US format with no way to tell. Verdict: "delegated level for internal tools with tests; would not use for anything parsing untrusted input."

Group B used the tool less successfully and understood it far better. Group B scores higher, and Group B's verdict is the one a team could actually act on.

## 🧭 Reflection

> You can now produce a working application faster than you can read one.
>
> For your capstone — and for the first job you take — where will you draw the line between the rows of the §1.1 table? Write the rule down now, before you are under deadline pressure, because that is when the line moves.

**Assessment.** This unit is graded as the **SE (Seminar)** component. Submit slides plus a one-page written evaluation by Nov 04. Individual contribution is peer-assessed within the group.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 24. Pearson.
- Chen, M., et al. (2021). _Evaluating Large Language Models Trained on Code._ arXiv:2107.03374.
- Perry, N., et al. (2023). _Do Users Write More Insecure Code with AI Assistants?_ ACM CCS.
- OWASP (2025). _Top 10 for Large Language Model Applications._
