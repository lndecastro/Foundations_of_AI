# Unit 10: Documents and Texts

```text
Session 16 | Oct 12 | HW10 (Office Portfolio) assigned Oct 12, due Oct 21
```

Part IV covers the three document types that consume most professional working hours: text, slides, and spreadsheets. This is the part of the course students most often expect to be trivial, and it is where the largest immediate time savings actually live.

For software engineers specifically, writing is not a side activity. Design documents, pull request descriptions, incident post-mortems, API documentation, and commit messages are the artifacts through which engineering work becomes legible to other people. They are also, uniformly, the thing engineers report disliking most. That combination — high volume, high value, low enthusiasm — is exactly where these tools earn their place.

## Learning Objectives

After completing this unit, you will be able to:

- Apply Unit 7's prompt structure to five recurring professional document types.
- Use AI to **interrogate** a long document rather than merely summarize it.
- Apply a verification standard proportional to each document's error cost.
- Identify what must never be pasted into a third-party tool.

## Part I — What AI Does Well with Text, and What It Does Not

| Genuinely useful | Requires care | Do not delegate |
| :--- | :--- | :--- |
| Restructuring your rough draft | Summarizing something you have not read | Deciding what the document should claim |
| Adjusting register for an audience | Any output containing numbers or citations | Judgments about people |
| Generating a first outline | Technical accuracy on unfamiliar material | Anything you will sign your name to unread |
| Finding what a long document says about X | Tone in emotionally loaded situations | The actual analysis in an incident review |

The organizing principle: **AI is strong at transformation and weak at commitment.** Give it your thinking and ask it to reshape that, and it performs well. Ask it to supply the thinking, and you get the most statistically typical version of the document — which is by definition unremarkable, and possibly wrong.

## Part II — Five Recurring Engineering Documents

Each entry below is a prompt skeleton in the Unit 7 six-component form. These belong in your prompt library.

### 2.1 Design Document

```text
You are a senior engineer turning rough notes into a design document for team review.

Below are my notes. Convert them into a design document.

Rules: Do not invent requirements, constraints, or options I did not state. Wherever my notes are vague, incomplete, or contradictory, insert [UNRESOLVED: what is missing] rather than filling the gap. I would rather have a document with eight unresolved markers than a smooth document that hides where my thinking stopped.

Format: Problem | Constraints | Options considered | Recommendation | Risks | Open questions.

Criteria: A reviewer should be able to disagree with the recommendation on specific technical grounds. If my notes do not contain enough to support a recommendation, say so instead of producing one.

MY NOTES: <paste your notes, however rough>
```

**Verify:** count the `[UNRESOLVED]` markers. If there are none, it smoothed over your gaps — ask again, more forcefully.

The `[UNRESOLVED]` instruction is the important one. Left alone, the model will close every gap in your thinking, producing a document that reads as finished and hides exactly the places where you needed help.

### 2.2 Pull Request Description

```text
You are writing a PR description for a reviewer with no context on this change.

Rules: Describe only what the diff shows. Do not claim the change is tested, benchmarked, or backwards-compatible unless I stated it. Do not describe the change as "simple," "minor," or "straightforward" — the reviewer decides that.

Format:
What changed — two or three sentences.
Why — the problem this solves.
How to review — where to look first, and what to check.
Not covered — what this PR deliberately does not do.

Criteria: A reviewer should know within fifteen seconds where to start reading.

DIFF SUMMARY: <files changed and a description of each>
CONTEXT THE DIFF DOES NOT SHOW: <issue number, prior discussion, the constraint you worked under>
```

**Verify:** read "Not covered" against your own knowledge of the change. It is the section most likely to come back thin, and the one that saves a reviewer an hour.

### 2.3 Incident Post-Mortem

```text
You are structuring a blameless post-mortem from a raw incident timeline.

Rules, and these are strict:
- Name no individual. Refer to roles.
- Include only causes the timeline actually supports. If you can see a likely cause the timeline does not establish, put it under "Hypotheses requiring investigation" — never under contributing factors.
- Distinguish *what happened* from *why it was possible*.
- Propose no action items I did not list. If you think one is missing, say so at the end under "Action items you may be missing."

Format: Impact | Timeline | Contributing factors | Hypotheses requiring investigation | What went well | Action items | Action items you may be missing.

Criteria: every contributing factor must point at a system or a process, not a person or a decision someone made under pressure.

RAW TIMELINE: <paste your Slack log, ticket history, or notes>
```

**Verify:** for each contributing factor, find the line in the raw timeline that establishes it. Any factor you cannot trace is a fabrication.

```{warning}
Post-mortems are the highest-risk document in this list for AI assistance. The model will readily generate plausible-sounding root causes that were never established, and a fabricated contributing factor sends remediation effort in the wrong direction. The "Hypotheses requiring investigation" section exists to give those guesses somewhere to go that is not the causes list.
```

### 2.4 Technical Documentation

```text
You are documenting a module for an engineer who knows the language but has never seen this codebase.

Rules: Document only behavior visible in the code below. Where you infer intent from a name rather than from logic, mark it [INFERRED]. Where the code's behavior on an edge case is genuinely unclear from reading it, mark it [UNCLEAR] rather than guessing.

Format: Purpose | Parameters | Returns | Raises | Example call | Caveats.

Criteria: name every assumption the code makes about its inputs — encoding, ordering, nullability, size, type. These are what break for the next person.

CODE: <paste the function or module>
```

**Verify:** check each `[INFERRED]` against the actual logic, and run the example call.

### 2.5 Difficult Email

```text
You are helping me write a professional email.

Rules: Do not apologize for anything that is not my fault. Do not overstate certainty. Do not use "I just wanted to," "circling back," or "per my last email." Do not invent an excuse or a reason I did not give you.

Format: subject line, then under 150 words of body.

Criteria: the recipient should finish reading knowing exactly what I am asking them to do and by when.

SITUATION: <what has happened>
RELATIONSHIP: <who they are to you, and the history>
WHAT I WANT TO HAPPEN: <the outcome, not the words>
WHAT I DO NOT WANT: <e.g. I do not want to escalate to their manager yet>
```

**Verify:** read it as the recipient. If you would feel managed rather than addressed, the register is wrong.

```{note}
Note the structural difference in 2.5: the field is WHAT I WANT TO HAPPEN, not what I want it to say. "Write an email declining this" produces something generic. "I need to decline without closing the door on future collaboration, and I do not want to invent an excuse" produces something usable, because you have specified the constraint that was actually hard.

All five prompts above appear in **Appendix D.4** alongside a sixth for interrogating long documents.
```

## Part III — Interrogating a Long Document

Most people ask for a summary. Summaries are the least useful thing to ask for, because they discard exactly the specificity you needed and you have no way to tell what was dropped.

Better patterns:

- **Targeted extraction.** *"What does this specification say about error handling? Quote the relevant passages and give section numbers. If it says nothing, say so."*
- **Contradiction hunting.** *"Identify places where this document contradicts itself or leaves a requirement ambiguous."*
- **Question answering with refusal.** *"Answer only from this document. If the answer is not present, say 'not addressed' and stop."*
- **Comparison.** *"What changed between v1 and v2 that would require a code change?"*

Each of these produces something checkable. A summary does not.

```{warning}
Never forward a summary of a document you have not read. If the summary is wrong, you have laundered a fabrication into your team's shared understanding with your name on it. Ask targeted questions, verify the answers against the source, and read the sections that matter.
```

## ⚙️ In-Class Activity: Five Scenarios

Working in groups of three, roughly 35 minutes. Each group takes one scenario, produces the document, then hands it to another group to critique.

**Scenario 1 — The vague design doc.** You have twelve lines of notes about choosing between polling and webhooks for a integration. Produce a design document that makes your uncertainty *visible* rather than hidden.

**Scenario 2 — The unreviewable PR.** A 400-line diff touching four files with the description "fixes bug." Write the description that should have been there.

**Scenario 3 — The incident.** A service was down for 40 minutes. You have a raw Slack timeline. Produce a blameless post-mortem — and identify at least one place where the AI proposed a cause the timeline does not support.

**Scenario 4 — The undocumented module.** A function with no docstring and non-obvious input assumptions. Document it, marking every `[INFERRED]` claim, then verify each one against the code.

**Scenario 5 — The hard email.** Your teammate has missed three agreed deadlines and the sprint is at risk. You need this addressed without escalating to a manager yet.

**Critique round.** The receiving group answers three questions: What in this document is asserted but not established? What did the author probably not know that the document conceals? Would you act on this?

## Part IV — Confidentiality

This is the responsibility beat for Unit 10, and it is the one most likely to matter to you during an internship.

Documents are the highest-risk category for accidental disclosure, because the useful thing to do with a document is paste the whole thing.

**Before pasting any document, ask:**

1. **Whose document is it?** Employer, client, and university documents are frequently not yours to share, regardless of whether you wrote them.
2. **Does it name people?** Names plus any attached fact — performance, health, grades, conduct — is personal data.
3. **Does it contain credentials?** Config files, connection strings, and stack traces routinely embed secrets. Stack traces in particular are pasted constantly and are full of internal paths and hostnames.
4. **What is the retention policy of the tool I am about to use?**

The safe habit is **redact before paste**: replace names, keys, hostnames, and identifiers with placeholders. It takes thirty seconds and it converts an irreversible mistake into a non-event.

## 💡 Example: A Summary That Cost a Sprint

An engineer asked an assistant to summarize a 60-page client specification and reported to the team that authentication was "OAuth 2.0 with standard scopes." The specification did say that — in section 4. Section 11, in a table the retrieval never surfaced, added a mandatory client-certificate requirement.

The summary was not false. It was **incomplete in a way the summary itself could not disclose**, because a summary's job is to omit and it has no way to tell you whether what it omitted mattered.

Targeted extraction — *"list every authentication requirement in this document with section numbers"* — would have surfaced both. The difference is not the model. It is the question.

## 🧭 Reflection

> You can now produce a competent design document in four minutes instead of forty. The four-minute version reads better than what you would have written.
>
> The forty-minute version had a property the four-minute one lacks: writing it forced you to discover what you did not yet know. What is your plan for keeping that, now that the friction that produced it is gone?

**Connecting to HW10 (Office Portfolio, Units 10–12):** this homework spans all three Part IV units and is due Oct 21. For Unit 10, submit **two documents** from the scenario list, each with: your prompt, the raw output, your edited final version, and a note on what you changed and why. The edits are what is graded.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 24. Pearson.
- Google (2024). _Site Reliability Engineering: Postmortem Culture._ <https://sre.google/sre-book/postmortem-culture/>
- Dendritic Institute (2025). _AI Literacy Series — Module 2, Part I: AI for Emails and Written Communication._
- Dendritic Institute (2025). _AI Literacy Series — Appendix: In-Class Group Scenarios._
