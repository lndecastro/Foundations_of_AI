# Unit 8: Careers in AI

> **Sessions 20–21** · Oct 26, Oct 28 · **HW8 assigned Oct 28, due Nov 04**
> *Session 21 includes a Guest Speaker / Industry Panel and the release of Capstone Guidelines.*

The most persistent misconception about AI careers is that they all require a doctorate in mathematics. The reality is that the majority of AI-related roles being created today are **not** research positions, and many of the most valuable ones sit at the boundary between technical systems and the domains they affect.

## Learning Objectives

After completing this unit, you will be able to:

- Map the **landscape of AI roles**, technical and non-technical.
- Identify the **skill clusters** each role family requires.
- Assess your own position against a target role and identify concrete gaps.
- Articulate how **domain expertise** functions as an AI career asset rather than a deficit.

## Part I — The Role Landscape

### 1.1 Building the Systems

| Role | What they do | Typical background |
| :--- | :--- | :--- |
| **ML Engineer** | Build, train, deploy models in production | CS, software engineering |
| **Data Scientist** | Analyze data, build models, communicate findings | Statistics, CS, quantitative fields |
| **Data Engineer** | Build the pipelines that feed models | Software engineering, databases |
| **Research Scientist** | Develop new methods | Ph.D., publication record |
| **MLOps Engineer** | Monitor, version, and maintain deployed models | DevOps, infrastructure |

### 1.2 Directing and Governing the Systems

| Role | What they do | Typical background |
| :--- | :--- | :--- |
| **AI Product Manager** | Decide what gets built and for whom | Business, domain expertise, PM |
| **AI Ethicist / Responsible AI Lead** | Assess harms, set policy, run reviews | Philosophy, law, social science |
| **AI Policy Analyst** | Shape regulation and compliance | Public policy, law, economics |
| **AI Auditor** | Test systems for bias, safety, compliance | Audit, risk, statistics |
| **Prompt / Context Engineer** | Design interactions with generative systems | Writing, linguistics, domain expertise |

### 1.3 Applying the Systems Within a Domain

This is the fastest-growing and least-recognized category — and for most students in this course, the most realistic entry point.

- **Clinical informatics specialist** — nursing or medicine plus AI fluency
- **Legal technologist** — law plus AI fluency
- **AI-literate educator, marketer, analyst, journalist, operations lead**

> **The pattern to notice:** these roles are not "AI person learns healthcare." They are **"healthcare person learns AI."** Domain expertise is expensive and slow to acquire; AI fluency is faster. If you already have or are building deep knowledge in a field, that is an asset, not a handicap.

## Part II — Skill Clusters

Rather than a list of job titles, think in terms of four transferable clusters:

| Cluster | Contents | Where it is learned |
| :--- | :--- | :--- |
| **Technical** | Python, statistics, ML fundamentals, data handling | Coursework, projects |
| **Data** | Quality assessment, privacy, provenance, governance | Unit 9 of this course |
| **Critical** | Evaluating claims, spotting failure modes, interrogating metrics | Units 5, 10, 11 |
| **Communication** | Explaining systems to non-experts, writing, presenting | Your capstone |

```{note}
Notice how much of this course maps onto clusters 2, 3, and 4. That is deliberate. Cluster 1 is the easiest to acquire on your own and the most commoditized. The others are what distinguish someone who can be trusted with an AI system from someone who can merely operate one.
```

## ⚙️ Hands-On: Skill Gap Analysis

This activity turns career planning into something concrete. Edit your own self-assessment, then see where you stand relative to target roles.

```python
import numpy as np, pandas as pd, matplotlib.pyplot as plt

SKILLS = ["Python / coding", "Statistics", "ML fundamentals",
          "Data handling", "Domain expertise", "Ethics & policy",
          "Communication", "Project management"]

# --- Typical skill demands by role (0 = not needed, 5 = essential) ---
roles = pd.DataFrame({
    "ML Engineer":        [5, 4, 5, 4, 2, 2, 3, 2],
    "Data Scientist":     [4, 5, 4, 5, 3, 2, 4, 3],
    "AI Product Manager": [2, 3, 3, 3, 5, 4, 5, 5],
    "AI Ethicist":        [1, 3, 3, 3, 4, 5, 5, 3],
    "Domain Specialist":  [2, 2, 3, 3, 5, 4, 4, 3],
}, index=SKILLS)

# --- EDIT THIS: rate yourself honestly, 0 to 5 ----------------------
me = pd.Series([2, 3, 2, 2, 4, 3, 4, 3], index=SKILLS, name="You")
# --------------------------------------------------------------------

TARGET = "AI Product Manager"      # <-- change to your target role

gap = (roles[TARGET] - me).clip(lower=0).sort_values(ascending=False)
print(f"Target role: {TARGET}\n")
print("Largest gaps to close:")
for skill, g in gap[gap > 0].items():
    print(f"  {skill:22s} gap of {g}  {'▓' * int(g)}")
if gap.max() == 0:
    print("  No gaps — consider a more demanding target role.")

# --- Radar chart: you vs. the target role ---------------------------
angles = np.linspace(0, 2 * np.pi, len(SKILLS), endpoint=False).tolist()
angles += angles[:1]

fig, ax = plt.subplots(figsize=(7.5, 7.5), subplot_kw=dict(polar=True))
for series, color, label in [(roles[TARGET], "darkgreen", TARGET),
                             (me, "darkorange", "You")]:
    vals = series.tolist() + series.tolist()[:1]
    ax.plot(angles, vals, color=color, linewidth=2, label=label)
    ax.fill(angles, vals, color=color, alpha=0.15)

ax.set_xticks(angles[:-1]); ax.set_xticklabels(SKILLS, fontsize=9)
ax.set_ylim(0, 5); ax.set_yticks(range(6))
ax.set_title(f"Your profile vs. {TARGET}", size=14, weight="bold", pad=25)
ax.legend(loc="upper right", bbox_to_anchor=(1.25, 1.1))
plt.tight_layout(); plt.show()
```

**Try changing it:**

1. Run it for all five roles. Which target requires the **least** movement from where you are now? That is often a better first job than the one you find most impressive.
2. Add a row for a skill this list is missing that matters in your field.
3. Look at where your profile *exceeds* the target. Those are your differentiators — the things you bring that a typical candidate does not.

```{important}
**On the "least movement" question.** Students routinely aim for ML Engineer because it sounds like the real AI job. But a student with strong domain knowledge, good communication, and moderate technical skill is far better positioned as a Product Manager or Domain Specialist — and those roles are frequently better paid, more durable, and harder to automate. Choose based on your actual profile, not on job-title prestige.
```

## 💡 Example: Two Paths to the Same Company

**Path A** — CS major → internship → ML Engineer. Builds the model.

**Path B** — Nursing major + this course + a health informatics certificate → Clinical AI Specialist. Decides *which* model should be built, evaluates whether it works on this hospital's patients, and is the person clinicians trust when it fails.

Path B is not a lesser version of Path A. In many organizations it is the more senior and less replaceable position, because it requires knowledge that cannot be acquired quickly.

## 🧭 Reflection

> Every role above involves working alongside systems that will keep changing. Which of your skills would survive the tools changing completely?
>
> If AI fluency becomes as common as spreadsheet fluency, what will differentiate professionals in your field?

**Connecting to HW8 (Career Exploration — Research an AI Role, due Nov 04):** pick a real, currently-posted job listing rather than a generic title. Analyze its actual stated requirements against your skill profile from the activity above. Bring questions for the Session 21 industry panel.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _AI: A Modern Approach_, 4th Ed., Ch. 28. Pearson.
- World Economic Forum. _Future of Jobs Report_ (most recent edition).
- Dendritic Institute (2025). _AI Literacy Series — Module 7: Foundational Models and Tools._
