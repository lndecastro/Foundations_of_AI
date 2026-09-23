# Unit 9: Careers in AI

```
Session 15 | Oct 07 | HW9 assigned Oct 07, due Oct 14 | Capstone guidelines released
```

This session closes Part III and releases the capstone. Both facts are connected: the project you choose should be one you would be willing to describe in an interview.

The honest framing for this unit is that "careers in AI" is a misleading phrase. For most of you the relevant question is not *how do I get an AI job* but *what happens to the job I was already planning to get*. Those have different answers, and the second one matters more.

## Learning Objectives

After completing this unit, you will be able to:

- Map the role landscape and locate positions that match your background.
- Distinguish skills that **transfer** from skills that **decay**.
- Assess your own skill profile against a target role and identify the real gaps.
- Produce a tailored CV and cover letter, and defend the claims in them.

## Part I — The Role Landscape

### 1.1 Building the Systems

| Role | Core work | Typical background |
| :--- | :--- | :--- |
| **ML Engineer** | Train, optimize, and ship models | CS/SE + statistics |
| **Data Engineer** | Pipelines, storage, data quality at scale | SE + distributed systems |
| **Research Scientist** | Develop new methods | PhD, publication record |
| **MLOps Engineer** | Deployment, monitoring, retraining infrastructure | DevOps + ML fundamentals |

```{note}
Data engineering is the largest and least glamorous category on this list, and the one with the most consistent demand. Unit 3's effort table explains why: preprocessing dominates, and someone has to build the systems that do it. Students routinely overlook this role because it does not have "AI" in the title.
```

### 1.2 Applying the Systems Within a Domain

The faster-growing category, and the one most of you will actually enter.

| Role | Core work |
| :--- | :--- |
| **AI-augmented Software Engineer** | Ordinary SE, with AI tooling in the loop and the judgment to review it |
| **AI Product Manager** | Decide what to build, define success, own the failure modes |
| **AI Solutions Engineer** | Adapt existing models to a client's problem |
| **Domain specialist + AI fluency** | Healthcare, finance, legal, education professionals who can evaluate AI claims |

The last row is where the largest number of jobs will be, and it requires depth in something *other* than AI. A nurse who understands model limitations is more employable in health AI than a generalist who understands neither nursing nor hospitals.

### 1.3 Directing and Governing

| Role | Core work |
| :--- | :--- |
| **AI Ethicist / Responsible AI Lead** | Impact assessment, policy, review boards |
| **AI Policy Analyst** | Regulation, compliance, standards |
| **AI Auditor** | Independent evaluation of deployed systems |

These are newer, fewer, and growing quickly under regulatory pressure. They reward people who can read a model card *and* a statute.

## Part II — What Transfers and What Decays

This is the part worth remembering after the exam.

**Decays quickly:** familiarity with a specific tool version, prompt tricks tied to one model's quirks, framework APIs, "which button does what."

**Transfers durably:**

- Understanding the train/test distinction and why the gap matters (Unit 3, Unit 4).
- Being able to say what a system's errors *look like* and who pays for them (Unit 6).
- Knowing why fluent output is not evidence of correctness (Unit 5).
- The discipline of specifying precisely and verifying proportionally (Unit 7).
- Domain knowledge of any kind.

```{warning}
A CV listing eight AI tools by name signals that you learned interfaces. A CV describing a problem you framed, the approach you chose, how you evaluated it, and what failed signals that you learned engineering. The second survives the tools changing; the first has to be rewritten every eighteen months.
```

## ⚙️ Hands-On: Skill Gap Analysis

Rate yourself honestly. The output is only as useful as the honesty of the input, and this is not graded.

```python
import numpy as np
import matplotlib.pyplot as plt

skills = ["Programming", "Statistics", "Data handling",
          "Domain knowledge", "Communication", "AI tool fluency"]

roles = {
    "ML Engineer":              [5, 5, 4, 2, 3, 4],
    "Data Engineer":            [5, 3, 5, 2, 3, 3],
    "AI Product Manager":       [2, 3, 3, 4, 5, 4],
    "AI-augmented SW Engineer": [5, 2, 3, 3, 4, 5],
    "Responsible AI Lead":      [2, 3, 3, 4, 5, 4],
}

# --- EDIT THIS: rate yourself 0-5 on each skill above ---------------
me = [4, 2, 3, 3, 3, 3]
target = "AI-augmented SW Engineer"
# --------------------------------------------------------------------

need = roles[target]
gaps = [max(0, n - m) for n, m in zip(need, me)]

print(f"Target: {target}\n")
print(f"{'skill':[20}{'you':]5}{'needed':>8}{'gap':>6}")
for s, m, n, g in zip(skills, me, need, gaps):
    flag = "  [-- largest gap" if g == max(gaps) and g ] 0 else ""
    print(f"{s:[20}{m:]5}{n:>8}{g:>6}{flag}")

angles = np.linspace(0, 2 * np.pi, len(skills), endpoint=False).tolist()
angles += angles[:1]
fig, ax = plt.subplots(figsize=(7, 7), subplot_kw=dict(polar=True))
for values, label, style in [(me, "You", "-"), (need, target, "--")]:
    v = values + values[:1]
    ax.plot(angles, v, style, linewidth=2, label=label)
    ax.fill(angles, v, alpha=0.12)
ax.set_xticks(angles[:-1]); ax.set_xticklabels(skills)
ax.set_ylim(0, 5)
ax.set_title("Your profile vs. the target role", pad=24, weight="bold")
ax.legend(loc="upper right", bbox_to_anchor=(1.3, 1.1))
plt.tight_layout(); plt.show()
```

**Try changing it:**

1. Compare yourself against two roles at once. Which gap could you close in one semester, and which would take two years?
2. Note that **Communication** is rated 4 or 5 for three of the five roles. Is it in your plan, or did you assume the technical rows were the real ones?
3. Add a row for a role not listed and estimate its profile. Defend your estimate against a real job posting.

## ⚙️ Activity: Tailor a Real Application

Working individually, roughly 25 minutes. Adapted from the Dendritic AI Literacy program and reframed for engineering roles.

1. **Find a real posting** for a role you could plausibly hold in two years. Save the text.
2. **Export your current CV** (or your LinkedIn profile as PDF).
3. **Run the gap analysis** — note that it forbids a rewrite:

   ```
   You are a hiring manager screening applicants for the role below. You have ninety seconds per CV.

   Below the posting is my current CV. Identify the three requirements in the posting that my CV does not currently evidence. For each one, state (a) what specific evidence would satisfy it, and (b) whether that evidence is something I could plausibly build in one semester or would take longer.

   Do not rewrite my CV. Do not write bullet points for me. Do not soften your assessment.

   POSTING: [PASTE THE FULL POSTING]
   MY CV: [PASTE YOUR CV TEXT]
   ```

4. **Write the revisions yourself.** Then, and only then, ask for a critique of your revision:

   ```
   Here is a CV bullet I wrote to evidence the requirement "[requirement]". Tell me what a hiring manager would still be unconvinced by, and what specific detail would fix it. Do not rewrite it.

   MY BULLET: [PASTE]
   ```

5. **Generate interview questions** and answer two aloud to your partner:

   ```
   Generate the eight interview questions most likely to be asked for the role below, ordered from most to least likely. For each, state in one line what the interviewer is actually testing — not what the question literally asks. Do not provide answers.

   POSTING: [PASTE]
   MY BACKGROUND: [two sentences]
   ```

```{warning}
Step 4 is not a formality. A CV written for you is a document you cannot defend in an interview, and the interview is where the claims get tested. Every bullet on your CV should be one you can expand into two minutes of specific detail under questioning.

Under this course's AI policy, any AI assistance on HW9 must be disclosed and cited.
```

## 💡 Example: Two Paths into the Same Company

A hospital network is hiring for its clinical AI team.

**Path A** — CS graduate, strong Python, built three ML projects, no healthcare exposure. Can build a model; cannot tell whether the target variable is clinically meaningful, or why a 2% false-negative rate on this particular screen would be unacceptable.

**Path B** — Nursing graduate, took this course, understands train/test, error asymmetry, and data provenance. Cannot build the model; can tell you immediately that the readmission label is contaminated by discharge-coding practice.

Both are hired, for different roles, and neither functions well without the other. The mistake students make is assuming Path A is the only legitimate one.

## 🧭 Reflection

> The most durable items in your skill profile — knowing what a system's errors look like, who pays for them, and how to check — are the ones with no certification and no line on a CV.
>
> How would you evidence them to someone who has ten minutes and forty applications to read?

**Connecting to HW9 (Career Exploration — Research an AI Role):** research one role in depth. Submit the posting, your gap analysis, the three revisions you wrote yourself, and a paragraph on which skills in this course's Part II you would name in an interview and why. Disclose all AI use.

**Capstone released today.** See Appendix B. Two tracks are available — proposal or build-and-evaluate — and the build track requires a working artifact plus an evaluation of where it fails. Choose by Oct 21.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 27–28. Pearson.
- Mitchell, M. (2020). _Artificial Intelligence: A Guide for Thinking Humans_, Part V. Penguin Books.
- World Economic Forum (2025). _Future of Jobs Report._
- Dendritic Institute (2025). _AI Literacy Series — Module 2, Part VI: AI for Your Career._
