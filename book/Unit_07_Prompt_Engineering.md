# Unit 7: Prompt Engineering and Context Engineering

```
Sessions 11–12 | Sep 23, 28 | HW7 assigned Sep 28, due Oct 05
```

Part III begins here. For six units you have studied what artificial intelligence is and how machines learn. From now on we focus on **using** generative AI well. Most of what you do in Units 8 through 15 rests on the skills developed in this unit.

The skill is simple to state and hard to master: an AI model responds based on what you tell it. Initially it does not know who you are, who the answer is for, what you already tried, or what "good" means to you. **Prompt engineering** is the practice of saying clearly what you want. **Context engineering** is the practice of giving the model everything it needs to know in order to deliver it.

This unit follows the same concepts and terminology as **Modules 5 and 6 of the Dendritic Institute AI Literacy Program**. If you complete that program, you will recognize every term used here.

## Learning Objectives

After completing this unit, you will be able to:

- Explain the purpose of prompt engineering and identify the four components of a prompt.
- Recognize and apply five common prompt types: instructional, role-based, chain-of-thought, zero-shot, and few-shot.
- Improve a prompt through iteration, changing one thing at a time.
- **Evaluate** a prompt-output pair using five quality criteria and check its factual claims.
- Use reverse prompting and meta-prompting to analyze, write, and improve prompts.
- Explain what context engineering is, how it differs from prompt engineering, and apply its main techniques.

---

## Part I — Prompt Engineering

### 1.1 What Is Prompt Engineering?

Prompt engineering is the practice of crafting effective inputs (prompts) for large language models (LLMs) so that they produce accurate, relevant, and useful outputs.

It is not programming. You communicate in natural language, but you do it **strategically**: every word you add either removes a guess the model would otherwise make, or it adds noise.

Compare these two requests:

```
Write something about hurricanes.
```

```
Write a one-paragraph explanation of how to prepare a home for a hurricane, for a family that has just moved to Florida from a state without hurricanes.
Use plain language and end with the single most important action to take before June 1.
```

The first prompt forces the model to guess the topic, the audience, the length, and the purpose. The second removes all four guesses. Neither prompt uses special syntax; the difference is entirely in how clearly the request is described.

### 1.2 Anatomy of a Prompt

A good prompt typically has four components. Only the first is always required, but you should know which ones you left out and why.

| Component | What it answers | Example |
| :--- | :--- | :--- |
| **1. Instruction** | What should the model do? | "Rewrite the instructions below in plain language." |
| **2. Context** (optional) | What background or perspective should it use? | "They are for my 82-year-old grandmother, who is not familiar with medical terms." |
| **3. Input Data** (optional) | What text, question, or file should it work on? | The pharmacy label text pasted below the request. |
| **4. Output Format** (optional) | What shape should the answer take? | "A numbered list of no more than five items, each under 12 words." |

Here is the complete prompt built from those four components:

```
Rewrite the pharmacy instructions below in plain language.
They are for my 82-year-old grandmother, who is not familiar with medical terms.
Present them as a numbered list of no more than five items, each under 12 words.

Instructions:
Take 1 tablet by mouth twice daily with food. Do not crush or chew.
Avoid grapefruit juice while taking this medication.
May cause drowsiness; use caution when driving or operating machinery.
Complete the full course even if symptoms improve.
```

```{note}
The component that beginners omit most often is **Context**. Without it, the model writes for an average reader of an average request, which is rarely the reader you have in mind. One sentence about who the answer is for often changes the output more than any other addition.
```

### Exercise 1: Deconstruct This Prompt

Identify the four components in the prompt below.

```
You are an assistant coach for a youth soccer team.
Using the attendance notes below, list the players who missed more than one practice this month.
Present the answer as a two-column table with the headings Player and Practices Missed.

Notes:
Ana missed March 3 and March 10. Ben missed March 3.
Carla missed March 3, March 10, and March 17.
Diego attended every practice. Emma missed March 17.
```

```{admonition} Check your answer
:class: dropdown
- **Instruction:** "list the players who missed more than one practice this month."
- **Context:** "You are an assistant coach for a youth soccer team." (a role that sets the perspective)
- **Input Data:** the attendance notes.
- **Output Format:** "a two-column table with the headings Player and Practices Missed."

The correct table contains Ana (2) and Carla (3) only. Run the prompt and check whether the model got it right; this is your first evaluation of an output.
```

### 1.3 Five Common Prompt Types

Prompt types (also called prompt patterns) are reusable structures. Knowing them lets you choose the right structure for your goal instead of starting from scratch every time.

**1. Instructional prompt.** Directly ask the model to perform a task.

```
Summarize the following paragraph in two sentences for a general audience.

The Atlantic hurricane season officially runs from June 1 to November 30.
Most storms form between August and October, when ocean temperatures are highest, and the statistical peak of the season is around September 10.
Forecasters issue a hurricane watch when hurricane conditions are possible within 48 hours and a hurricane warning when they are expected within 36 hours.
```

**2. Role-based prompt.** Assign the model a role or identity that shapes its tone and the knowledge it draws on.

```
You are a financial counselor at a university.
A first-year student asks how to start building credit without getting into debt.
Give practical advice in five bullet points.
```

**3. Chain-of-thought prompt.** Ask the model to work through the problem step by step before answering.

```
A family is driving 540 miles. Their car averages 30 miles per gallon, and gas costs $3.40 per gallon.
They will split the fuel cost equally with another family.
Work through the calculation step by step, then state how much each family pays.
```

The correct answer is \$30.60 (18 gallons × \$3.40 = \$61.20, divided by 2). Showing the steps lets you check each one, which is the main practical benefit of this pattern.

**4. Zero-shot prompt.** Give only the instruction and the input, with no examples.

```
Classify the following customer comment as positive, negative, or neutral.

Comment: "The delivery came a day late, but the flowers were beautiful."
```

**5. Few-shot prompt.** Give a few examples of input and output so the model can imitate the pattern.

```
Classify each customer comment.

Comment: "Fast shipping and great quality." → Positive
Comment: "The box was crushed and nothing worked." → Negative
Comment: "Great taste, but the portion was tiny." → Mixed
Comment: "The delivery came a day late, but the flowers were beautiful." →
```

Notice what the few-shot version did: the examples introduced a category, **Mixed**, that the zero-shot version never offered. Examples communicate things that are hard to describe in words, such as a label set, a tone, or a format.

```{note}
A prompt can belong to more than one type at the same time.
The instructional prompt above is also a zero-shot prompt, and the role-based prompt could become few-shot by adding examples of good advice.
```

```{note}
Many recent models reason step by step on their own before answering, so asking for chain-of-thought improves their answers less than it did with earlier models.
It is still valuable for a different reason: it makes the reasoning **visible**, so you can check it.
```

| Type | Best used for | Example |
| :--- | :--- | :--- |
| Instructional | Clear, single-step tasks | "Summarize this paragraph." |
| Role-based | Matching tone, perspective, or expertise | "You are a financial counselor…" |
| Chain-of-thought | Multi-step reasoning, calculations | "Work through it step by step…" |
| Zero-shot | Quick, common tasks | "Classify this comment…" |
| Few-shot | Custom formats, labels, or styles | Examples first, then the new case |

### 1.4 Iteration: The First Prompt Is a Draft

Even a good prompt can return a vague, incomplete, or wrong answer. Prompting is a loop, not a single attempt:

1. **Draft** the initial prompt.
2. **Review** the output. Is it accurate? Complete? In the right tone and format?
3. **Refine** the prompt. Add what was missing, clarify the goal, or adjust the role.
4. **Test again** and compare the new output with the previous one.

One rule makes this loop useful: **change one thing at a time**. If you change four things and the output improves, you do not know which change helped, so you have learned nothing you can reuse next time.

A second rule: **test each version in a new conversation**. The model can see everything earlier in the same conversation, so a previous attempt can influence the next one. Part III explains why.

## ⚙️ Hands-On 1: Improving a Prompt One Step at a Time

Open any AI assistant (ChatGPT, Claude, Gemini, Copilot, or another). Paste the prompt below into a **new conversation** and save the output.

```
Explain what a credit score is.
```

**Try changing it:**

1. **Add Context (audience).** In a new conversation, paste:

   ```
   Explain what a credit score is to a first-year college student who has never had a credit card.
   ```

   Compare the two outputs. What changed in the vocabulary, the length, and the examples used?

2. **Add Output Format.** In a new conversation, paste:

   ```
   Explain what a credit score is to a first-year college student who has never had a credit card.
   Use exactly four bullet points, then end with one everyday analogy. Keep the whole answer under 150 words.
   ```

   Did the model follow every format instruction? Count the bullet points and the words yourself; do not assume.

3. **Add a role and chain-of-thought.** In a new conversation, paste:

   ```
   You are a financial counselor at a university. Explain what a credit score is to a first-year college student who has never had a credit card.
   First, list the factors that make up a credit score.
   Then explain which of those factors a student can control during the first year of college, and give one concrete action for each.
   Keep the whole answer under 200 words.
   ```

   Which of the three changes produced the largest improvement for this student? Do you think the same change would matter most for a different task?

---

## Part II — Evaluating Prompts and Outputs

### 2.1 Why "It Looks Good" Is Not an Evaluation

In Unit 5 you saw that a language model generates the most plausible continuation of its input. Plausible and correct often coincide, but not always. The model writes a wrong answer with the same fluency and confidence as a right one. A **hallucination** is an output that states incorrect or invented information as if it were true.

Two further facts complicate evaluation:

- **The same prompt can produce different outputs.** Run it twice and you may get two different answers. One good output does not prove the prompt is good.
- **A well-formatted answer is not evidence of a correct one.** A neat table of numbers can contain numbers that were never calculated.

So evaluation must be deliberate. It has two parts: **scoring the output against criteria** and **checking its claims**.

### 2.2 Five Criteria for a Prompt-Output Pair

| Criterion | Question to ask | How to check it |
| :--- | :--- | :--- |
| **1. Relevance** | Does the response address what the prompt asked, for the audience it named? | Reread the instruction and context. Mark anything in the output that answers a different question. |
| **2. Completeness** | Does it cover every requested part? | List each item the prompt requested. Tick each one you find in the output. |
| **3. Clarity** | Is it easy to understand and well organized? | Could the intended reader act on it without asking a follow-up question? |
| **4. Factual Accuracy** | Are the claims and numbers correct and verifiable? | Check each factual claim against the source you provided, or an independent source you trust. |
| **5. Format** | Does it follow the requested structure, length, and style? | Count bullets, words, columns. Compare with what you asked for. |

Score each criterion from 1 to 5:

| Score | Meaning |
| :--- | :--- |
| **5** | Fully meets the criterion; nothing to fix. |
| **3** | Partly meets it; usable after edits. |
| **1** | Fails it; the output cannot be used as is. |

```{warning}
Do not simply add up the five scores. **Factual Accuracy is a gate, not an ingredient.** An output that scores 5 on everything except accuracy is not "80% good"; if you use it, you pass on an error. When accuracy fails, the output fails, whatever the total.
```

### 2.3 How to Evaluate: A Six-Step Procedure

1. **Decide what a good answer must contain before you run the prompt.** Write down the items you expect to see. If you cannot say what a good answer looks like, the prompt is not ready.
2. **Run the prompt in a new conversation.**
3. **Score the five criteria**, using the "How to check it" column above.
4. **Check every factual claim.** Go through the output sentence by sentence and label each claim as *supported*, *not supported*, or *cannot verify*.
5. **Run the prompt a second time**, in another new conversation. If the two outputs differ substantially, the prompt leaves too much to guesswork.
6. **Change one thing** in the prompt to address the weakest criterion, and repeat.

### 2.4 Worked Example

A student asks an AI assistant to summarize a library announcement for a neighborhood newsletter.

**The prompt:**

```
Summarize the library announcement below for a neighborhood newsletter.
Use exactly three bullet points.
Include the new hours, the laptop rules, and the cost of late returns.
Use only the information in the announcement.

Announcement:
Starting March 1, the Riverside Public Library will extend its weekday hours to 9 a.m. to 8 p.m.
Weekend hours remain 10 a.m. to 5 p.m. The library will also begin lending laptops to cardholders aged 18 and older.
Laptops may be borrowed for up to 7 days and cannot be renewed. Late returns are charged $5 per day.
The program is funded by a two-year state grant.
```

**Step 1, before running:** a good answer must contain (a) weekday hours, (b) weekend hours, (c) who can borrow laptops, (d) loan length and no renewal, (e) the \$5 daily late fee, in three bullets.

**The output (an illustrative example):**

> - Starting March 1, the Riverside Public Library is open longer, from 9 a.m. to 8 p.m. every day.
> - Adults can now borrow laptops for up to two weeks, thanks to a state grant.
> - Laptop loans are free, so stop by and pick one up!

**Step 4, claim check:**

| Claim in the output | Label | Reason |
| :--- | :--- | :--- |
| Open 9 a.m. to 8 p.m. from March 1 | Supported, partly | True for weekdays only. |
| …every day | **Not supported** | Weekend hours are unchanged. |
| Adults can borrow laptops | Supported | Cardholders aged 18 and older. |
| …for up to two weeks | **Not supported** | The limit is 7 days, with no renewal. |
| Funded by a state grant | Supported | Stated in the announcement. |
| Laptop loans are free | **Not supported** | Never stated, and misleading given the late fee. |

**Step 3, scores:**

| Criterion | Score | Reason |
| :--- | :---: | :--- |
| Relevance | 5 | Addresses the announcement for newsletter readers. |
| Completeness | 2 | Late fee missing; weekend hours missing. |
| Clarity | 5 | Easy to read. |
| Factual Accuracy | **1** | Two errors and one invented claim. |
| Format | 5 | Exactly three bullets. |

The total is 18 out of 25, which sounds acceptable. It is not. A reader following this newsletter would arrive on a Saturday evening to a closed library, keep a laptop for two weeks, and be surprised by a \$35 fine. **The accuracy gate fails, so the output fails.**

Notice also that the prompt already said "use only the information in the announcement". The model still added a claim. Instructions reduce errors; they do not eliminate them. That is why step 4 exists.

**Step 6, refining the prompt.** The weakest criterion is accuracy, and the errors cluster around hours and loan rules. One change:

```
Summarize the library announcement below for a neighborhood newsletter.
Use exactly three bullet points: one for the hours (list weekday and weekend hours separately), one for the laptop rules (who, how long, renewals), and one for late returns.
Use only the information in the announcement. Do not add benefits, prices, or rules that are not stated.

Announcement:
Starting March 1, the Riverside Public Library will extend its weekday hours to 9 a.m. to 8 p.m.
Weekend hours remain 10 a.m. to 5 p.m. The library will also begin lending laptops to cardholders aged 18 and older.
Laptops may be borrowed for up to 7 days and cannot be renewed. Late returns are charged $5 per day.
The program is funded by a two-year state grant.
```

The revision assigns each bullet a job, which makes omissions obvious, and it names the kind of addition that is forbidden. You would now run it, in a new conversation, and evaluate again.

### 2.5 Techniques to Reduce and Detect Hallucinations

- Tell the model to **use only the provided text or file**.
- Tell the model to say **"I do not know"** or "not stated" when the information is not available.
- Ask the model to **cite its sources**, and then open every source. A fabricated reference usually looks real: plausible authors, plausible journal, plausible year.
- Ask the model to **explain the data and generalizations** it presents, so you can see where a number came from.
- **Check numbers yourself.** A model can produce a statistic that appears in no source at all.

### 2.6 Troubleshooting a Prompt

| If the output is… | Try this |
| :--- | :--- |
| Too generic | Add Context: who it is for, and why they need it. Add an example. |
| Hallucinating | Restrict it to provided information; ask it to say "not stated" when unsure; ask for sources. |
| Missing parts | Give each part of the request its own numbered item or bullet. |
| Inconsistent between runs | Specify the role and the output format more precisely; add a few-shot example. |
| Too long | State a word or bullet limit, and say what to leave out. |

### 2.7 Reverse Prompting: Analyzing Prompts Backwards

Reverse prompting starts from an **output** and asks: *what prompt would have produced this?* It is not a prompt type; it is an analysis method. It trains you to see which features of a prompt produce which features of an answer.

You can do it yourself, or ask the model to do it with a **meta-prompt** (a prompt about prompts; Section 2.8 covers meta-prompting in full):

```
Here is an AI-generated output:

1. Check whether your homeowner's or renter's insurance covers flood damage.
2. Photograph your belongings and store the photos online.
3. Pack a go-bag with medications, documents, water, and chargers.

Your task is to reverse prompt this output.
Propose three different prompts that could have generated it.
For each one, explain which words in the prompt would produce the numbered format, the practical tone, and the focus on preparation before a storm.
Then explain how changing the audience in each prompt would change the output.
```

Compare the model's reconstructed prompts with your own guesses. Did either include a role? An audience? A number of items? Each feature you can explain is a feature you can now produce deliberately.

### 2.8 Meta-Prompting: Using AI to Write and Improve Prompts

A **meta-prompt** is a prompt whose subject is another prompt. Reverse prompting is one use of it: you give the model an output and ask for the prompt behind it. **Meta-prompting** is the broader practice of asking the model to help you **write, question, critique, or improve** a prompt before you use it.

It works because the model has seen a great many instructions and requests, so it is good at noticing what a request leaves unsaid. It cannot, however, know facts about your situation that you have not given it. Meta-prompting helps you find the gaps; only you can fill them.

**Use 1: Draft a prompt from a goal.** Describe what you want to achieve and ask the model to write the prompt.

```
I volunteer at an animal shelter.
Every week I write short online profiles for dogs available for adoption, and I want an AI assistant to draft them for me from my notes.
Write a prompt I can reuse for this task.
The prompt must contain the four components of a prompt: an instruction, context, a place for the input data, and an output format.
Explain in one sentence why you chose each component.
```

**Use 2: Let the model interview you first.** Ask the model to question you before it writes anything. This is the most effective way to discover the context you forgot to give.

```
I want to write a prompt that helps me prepare for a job interview for a part-time position at a local bank.
Before you write the prompt, ask me up to five questions about anything you would need to know to make it effective.
Ask the questions one at a time and wait for my answer to each. After the last answer, write the complete prompt.
```

**Use 3: Critique and improve an existing prompt.** Paste a prompt and ask the model to evaluate it against the criteria from Section 2.2 before you run it.

```
Here is a prompt I plan to use:

"Summarize the library announcement below for a neighborhood newsletter.
Use exactly three bullet points.
Include the new hours, the laptop rules, and the cost of late returns.
Use only the information in the announcement."

Do not answer the prompt. Instead, evaluate it as a prompt.
For each of these criteria, explain how the prompt could lead to a weak output: relevance, completeness, clarity, factual accuracy, and format.
Then write an improved version and list each change you made with the reason for it.
```

This is the first prompt from the worked example in Section 2.4. Compare the model's critique with what actually went wrong there: the merged weekday and weekend hours, the wrong loan period, and the invented "free" claim. Did the critique anticipate those failures, or only generic ones?

| | Reverse prompting | Meta-prompting |
| :--- | :--- | :--- |
| Starts from | An output | A goal or an existing prompt |
| Produces | Possible prompts that could have generated the output | A new or improved prompt, or questions about it |
| Main use | Understanding why prompts produce what they produce | Writing and improving your own prompts |

```{warning}
A prompt written by a model is still a **draft**, and it needs the same evaluation as any other prompt. Watch for three common problems. It may contain placeholders such as [insert details], which you must fill in or delete. It may add requirements you never wanted, such as a word count or a tone. And it can only include the context you supplied, so a polished-looking prompt can still be missing the one fact that matters. Run the improved prompt, evaluate its output with Section 2.3, and compare it with your own version before you adopt it.
```

### Exercise 2: Meta-Prompting

1. Choose a task you actually do: writing to a professor, planning a trip, preparing a presentation, or summarizing readings.
2. Write your own prompt for it first, without help.
3. In a new conversation, use the interview-first meta-prompt (Use 2), adapted to your task, and answer the model's questions.
4. Run both prompts, each in its own new conversation, and score both outputs with the five criteria.
5. Which prompt won? Which questions from the model revealed context you had left out?

### 2.9 How Much Checking Is Enough?

Not every output deserves the same scrutiny. Match the effort to the cost of an error.

| Cost of an error | Example | What to check |
| :--- | :--- | :--- |
| **Trivial** | Brainstorming names for a club | Read it. |
| **Low** | A first draft of an email | Check any fact, date, or name it contains. |
| **Moderate** | A summary you will forward to others | Check every claim against the source. |
| **High** | Anything with a citation, statistic, or legal or medical statement | Verify each one independently. |
| **Severe** | Decisions about someone's health, money, safety, or record | Use AI for support at most; the judgment stays with a qualified person. |

This table is also the reasoning behind the course AI Use Statement: you may use AI on homework, but you are responsible for what you submit.

## ⚙️ Hands-On 2: Evaluate an Output

Paste the prompt below into a new conversation. Before reading the output, write down what a good answer must contain (step 1 of the procedure). Then complete steps 3 and 4: score the five criteria and build a claim-check table.

```
You are a student-services advisor.
Read the scholarship notice below and turn it into a checklist a student can follow to apply.
Present a numbered checklist, then one line stating the deadline. Use only the information in the notice.
If a student would need to know something that the notice does not state, list it under the heading "Not stated: ask the Fund".

Notice:
The Harbor Valley Community Fund Scholarship awards $2,500 to one student each year.
Applicants must be enrolled full time at a public college or university in Florida and have a cumulative GPA of at least 3.0.
Applicants must submit a 500-word essay describing a problem in their local community and how they would address it, plus two letters of recommendation, at least one of which must come from a faculty member.
Applications are due November 15 at 5:00 p.m. Finalists will be interviewed in December, and the recipient will be announced on January 20.
```

**Try changing it:**

1. **Remove the safeguards.** In a new conversation, run the same prompt but delete the two sentences that begin "Use only" and "If a student". Build a new claim-check table. Did any unsupported claims appear, such as a submission address, a required transcript, or a renewal rule?
2. **Test consistency.** Run the original prompt again in another new conversation. Did the checklist order, the number of items, or the "Not stated" list change? What does that tell you about how much one output can prove?
3. **Reverse prompt it.** In a new conversation, paste your best output from this exercise and ask the model to reverse prompt it, using the meta-prompt from Section 2.7 as a model. Did the reconstructed prompts include your "Not stated" instruction? If not, what does that tell you about which parts of a prompt leave visible traces in the output?

---

## Part III — Context Engineering

### 3.1 What Is Context Engineering?

**Context engineering** is the practice of designing, structuring, and managing everything the model can see when it produces an answer, not just the instruction itself.

- Prompt engineering asks: *how should I phrase the request?*
- Context engineering asks: *what information should surround the request so that the model can answer it well?*

It is sometimes called "prompt engineering on steroids" because it moves from a single well-worded request to deliberate control of the model's whole working environment.

### 3.2 Why Context Matters: The Context Window

A language model has no memory of you, your class, or your job. Each time it answers, it reads only its **context window**: everything present in the current exchange. That includes:

- any instructions set by the application or by you (for example, custom instructions),
- the earlier messages in the same conversation,
- any files or images you attached,
- and your latest message.

Nothing outside that window influences the answer. This has four practical consequences.

1. **What you do not include, the model does not know.** If your budget is \$60, and you do not say so, the model plans for an average budget.
2. **Relevant material improves answers.** Supplying the actual document reduces hallucination, because the model can draw on the text instead of on vague patterns.
3. **Irrelevant material makes answers worse.** Pasting in five loosely related documents "just in case" gives the model more to be distracted by. More context is not automatically better context.
4. **Earlier messages stay in the window.** Suppose that ten messages ago you asked for recipes that include peanuts, and later you say you are cooking for a friend with a peanut allergy. The earlier request is still in the context, competing with the correction. When a conversation has gone wrong, **starting a new conversation with a clean, complete prompt usually works better than correcting it message by message.**

By shaping the context deliberately, you can steer the model toward the answer you need, ground it in reliable sources, fit it to a specific role, and make the same task repeatable.

### 3.3 Key Techniques in Context Engineering

**1. System role design.** Define the model's role and purpose in detail: who it is, whom it serves, and how it should communicate. This goes further than a one-line role.

```
You are a museum educator at a natural history museum. You help visiting families with children aged 8 to 12 understand the exhibits. Use short sentences, one surprising fact per answer, and a question the children can discuss afterward. Avoid scientific terms unless you explain them in the same sentence.

Explain why dinosaurs are considered relatives of modern birds.
```

**2. Instruction layering.** Combine several patterns in one request: a role, a task, a format, and a goal.

```
Act as a travel planner for students on a tight budget. List five low-cost activities for a weekend in St. Augustine, Florida. Then write a 100-word plan for Saturday that uses three of them, with an estimated total cost.
```

**3. Contextual priming.** Provide reference material or an example that shapes how the model responds. Unlike few-shot prompting, which teaches a pattern through several input-output pairs, priming supplies a model to follow.

```
Below is an example of a strong restaurant review (Review 1). Rewrite Review 2 using the same structure: one sentence on atmosphere, one on a specific dish, one on service, and a final sentence with a clear recommendation.

Review 1: The dining room is small and warm, lit mostly by candles on each table. The grouper sandwich arrived with a crisp crust and a bright lime sauce that balanced the richness of the fish. Our server remembered our drink order without writing it down and checked on us exactly once. For a relaxed dinner under $25, this is the best choice in the neighborhood.

Review 2: It was ok. Food was kind of good I guess, we got pasta. The waiter was nice but slow. Probably would go again maybe.
```

**4. Retrieval-Augmented Generation (RAG).** In professional systems, RAG automatically **retrieves** relevant passages from a collection of documents and adds them to the context before the model **generates** its answer. The answer is then grounded in those passages rather than in the model's general training. When you attach a document to a chat and ask the model to answer from it, you are doing a simplified, manual version of the same idea.

Download the course syllabus from Canvas and attach it to this prompt:

```
You are demonstrating Retrieval-Augmented Generation for a classroom lesson.

Step 1 — Retrieve: From the attached syllabus only, find the passages that describe (a) the policy for late submissions and (b) the rules for using AI tools on the capstone project. Quote each passage word for word and name the section heading it appears under.

Step 2 — Label: Present those passages under the heading Retrieved Content. Do not paraphrase them and do not add anything that is not in the document.

Step 3 — Generate: Under the heading Generated Answer, write three sentences explaining what a student must do to submit a late assignment and to use AI on the capstone, using only the retrieved passages.
```

Then complete the two steps that only you can do. **Verify:** open the syllabus and confirm every quotation, noting any the model altered, merged, or invented. **Explain:** how is this different from contextual priming, where you paste the passage yourself and the model does no searching?

**5. Chained prompts.** Break a complex task into stages and send them as separate messages, checking each output before moving on. Errors caught at stage 1 do not propagate into stage 3.

```
Step 1: List five common reasons members cancel their membership at a small neighborhood gym.
```

After you review and correct that list, send:

```
Step 2: For each of those five reasons, propose one low-cost action the gym owner could take to keep those members.
```

Then:

```
Step 3: Summarize your recommendations in a 120-word memo to the gym owner, ordered from cheapest to most expensive.
```

**6. Output shaping.** Specify tone, style, length, and format for a specific destination.

```
Write a reminder to parents that the school science fair is next Thursday at 6 p.m. in the gym. Write it as a text message: friendly, under 50 words, with the date and time in the first sentence.
```

**7. Negative context engineering.** State explicitly what to avoid or leave out.

```
Explain three benefits of regular physical activity for college students. Do not recommend specific products, apps, or supplements. Do not use medical terminology. Do not give weight-loss targets.
```

**8. Multimodal context.** Combine different kinds of input, such as text and images, in one request. Take a photo of the nutrition facts label on any packaged food you have, attach it, and paste:

```
Using only the attached photo of a nutrition facts label, state the serving size, the number of servings in the package, and the calories in the whole package. Show the calculation for the whole-package calories. If any value is unreadable in the photo, say so instead of estimating.
```

Check the calculation against the label yourself.

### 3.4 Prompt Engineering vs. Context Engineering

| Aspect | Prompt Engineering | Context Engineering |
| :--- | :--- | :--- |
| Focus | Wording of the instruction | Everything around the instruction |
| Scope | A single prompt | Roles, reference materials, files, conversation history, sequences of prompts |
| Goal | A better single response | Consistent, reliable, domain-specific responses |
| Analogy | Writing a recipe | Designing the whole kitchen and stocking the pantry |

The two are not rivals. Every context-engineered input still contains a well-engineered prompt; context engineering adds what the prompt alone cannot carry.

### 3.5 Worked Example: From Prompt to Context

**Prompt engineering only:**

```
Plan dinners for next week.
```

The model must guess how many people, which days, what budget, what diet, what skills, and what is already in the kitchen. It will guess all of them, plausibly and probably wrongly.

**Context engineered:**

```
System Role: You are a practical home-cooking coach who helps college students eat well on a small budget.

Reference Context: I share an apartment with one roommate, who is vegetarian. We cook dinner together Monday through Friday. Our total grocery budget for the week is $60. We have one pot, one frying pan, and no oven. Already in our pantry: rice, dried black beans, pasta, olive oil, garlic, onions, and canned tomatoes.

Task: Plan five vegetarian dinners for two people that use the pantry items first. Then write one combined shopping list for the ingredients we still need to buy.

Constraints: Each dinner must take 30 minutes or less. Do not use any recipe that requires an oven. Do not exceed the budget; if prices are estimates, say so.

Output: A table with the columns Day, Dinner, Pantry Items Used, and Cooking Time, followed by the shopping list grouped by store section, with an estimated total cost.
```

| Line | Technique |
| :--- | :--- |
| System Role | System role design |
| Reference Context | Contextual priming: the facts the model could not know |
| Task (two parts) | Instruction layering |
| Constraints | Negative context engineering |
| Output | Output shaping |

Now evaluate it with Part II: is every dinner vegetarian, oven-free, and under 30 minutes? Does the shopping list actually cover every ingredient in the table? Is the total cost plausible, and did the model flag it as an estimate?

## ⚙️ Hands-On 3: From Prompt Engineering to Context Engineering

Paste this baseline prompt into a new conversation and save the output.

```
Summarize the following idea in three bullet points.

Idea: A phone app that alerts residents when local drainage canals are close to overflowing during heavy rain, using water-level sensors and weather forecasts.
```

**Try changing it:**

1. **Add a role and reference context.** In a new conversation, paste the version below. Score both outputs with the five criteria from Part II. Which criteria improved, and by how much?

   ```
   System Role: You are a city planning consultant who advises small cities on flood preparedness.

   Reference Context: The city has 40,000 residents and a limited technology budget. About one third of residents are over 65, and many of them do not use smartphones. Heavy summer rain closes several roads each year.

   Task: Summarize the idea below and assess whether it fits this city.

   Idea: A phone app that alerts residents when local drainage canals are close to overflowing during heavy rain, using water-level sensors and weather forecasts.

   Output: Three bullet points under the headings Value to Residents, Practical Obstacles, and Recommended Next Step.
   ```

2. **Add negative context.** Add these two sentences before the Output line and run it in a new conversation: *"Do not recommend any specific commercial product or company. Do not assume a budget larger than the one described."* What disappeared from the answer? Was anything useful lost?

3. **Remove one component.** Run version 1 again in a new conversation, but delete the Reference Context lines. Which of the three bullets changed the most? The component whose removal hurts the output most is the one doing the work, and it is the one you should never leave out for this kind of task.

---

## 💡 Looking Ahead

Writing the same role and reference context into every conversation is tedious. In **Unit 8**, you will save context so that it applies automatically: **personalized assistants** (such as Custom GPTs and Gems) store a role, instructions, and knowledge files for one purpose, and **AI workspaces** (Projects) keep files, instructions, and conversations together for ongoing work. Both are context engineering made permanent. They correspond to Section 6.6 of AI Literacy Module 6.

## 🧭 Reflection

1. How does communicating with an AI model differ from communicating with another person who shares your background?
2. Which prompt type felt most natural to you, and which needed the most experimentation?
3. In the worked example of Section 2.4, the prompt said "use only the information in the announcement" and the model still added a claim. What does that imply about how much any prompt can guarantee?
4. A well-engineered prompt makes the output more likely to be **what you asked for**. Does it make it more likely to be **correct**? Are those the same thing?

**Connecting to HW7 (Prompt Iteration Log):** choose one real task from your own life, studies, or intended career. Write an initial prompt and then five successive revisions, changing one thing each time and running each in a new conversation. For each version, record what you changed and why, the five criterion scores, and any claims you checked. At least one revision must come from a meta-prompt (Section 2.8); record what the model changed and which changes you kept. Your final version must use at least two context engineering techniques from Section 3.3; name them. End with the AI Disclosure statement required by the syllabus. The log is graded, not the final prompt.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 24. Pearson.
- Mitchell, M. (2020). _Artificial Intelligence: A Guide for Thinking Humans_, Ch. 11–13. Penguin Books.
- Mollick, E., & Mollick, L. R. (2024). _Co-Intelligence: Living and Working with AI_. Portfolio.
- Prompt Engineering Guide. [https://www.promptingguide.ai/](https://www.promptingguide.ai/)
- White, J. et al. (2023). _A Prompt Pattern Catalog to Enhance Prompt Engineering with ChatGPT_. arXiv:2302.11382. [https://arxiv.org/abs/2302.11382](https://arxiv.org/abs/2302.11382)
- Zhou, Y. et al. (2023). _Large Language Models Are Human-Level Prompt Engineers_. arXiv:2211.01910. [https://arxiv.org/abs/2211.01910](https://arxiv.org/abs/2211.01910)
- Wei, J. et al. (2022). _Chain-of-Thought Prompting Elicits Reasoning in Large Language Models_. arXiv:2201.11903. [https://arxiv.org/abs/2201.11903](https://arxiv.org/abs/2201.11903)
- Lewis, P. et al. (2020). _Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks_. arXiv:2005.11401. [https://arxiv.org/abs/2005.11401](https://arxiv.org/abs/2005.11401)
- Anthropic (2025). _Effective Context Engineering for AI Agents_. [https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- Dendritic Institute (2025). _AI Literacy Series — Module 5: Fundamentals of Prompt Engineering_ and _Module 6: Context Engineering_.
