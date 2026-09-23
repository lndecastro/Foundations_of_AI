# Unit 7: Prompt Engineering and Context Engineering

```
Sessions 11–12 | Sep 23, 28 | HW7 assigned Sep 28, due Oct 05
```

Part III begins here. For six units you have studied what artificial intelligence is and how machines learn. From now on we focus on **using** generative AI well. Most of what you do in Units 8 through 15 rests on the skills developed in this unit.

The skill is simple to state and hard to master: an AI model only knows what you tell it. It does not know who you are, who the answer is for, what you already tried, or what "good" means to you. **Prompt engineering** is the practice of saying clearly what you want. **Context engineering** is the practice of giving the model everything it needs to know in order to deliver it.

This unit follows the same concepts and terminology as **Modules 5 and 6 of the Dendritic Institute AI Literacy Program**. If you complete that program, you will recognize every term used here.

## Learning Objectives

After completing this unit, you will be able to:

- Explain the purpose of prompt engineering and identify the four components of a prompt.
- Recognize and apply five common prompt types: instructional, role-based, chain-of-thought, zero-shot, and few-shot.
- Improve a prompt through iteration, changing one thing at a time.
- **Evaluate** a prompt-output pair using five quality criteria and check its factual claims.
- Explain what context engineering is, how it differs from prompt engineering, and apply its main techniques.

---

## Part I — Prompt Engineering

### 1.1 What Is Prompt Engineering?

Prompt engineering is the practice of crafting effective inputs (prompts) for large language models (LLMs) so that they produce accurate, relevant, and useful outputs.

It is not programming. You communicate in ordinary language, but you do it **strategically**: every word you add either removes a guess the model would otherwise make, or it adds noise.

Compare these two requests:

```
Write something about hurricanes.
```

```
Write a one-paragraph explanation of how to prepare a home for a hurricane, for a family that has just moved to Florida from a state without hurricanes. Use plain language and end with the single most important action to take before June 1.
```

The first prompt forces the model to guess the topic, the audience, the length, and the purpose. The second removes all four guesses. Neither prompt uses special syntax; the difference is entirely in how clearly the request is described.

### 1.2 Anatomy of a Prompt

A good prompt typically has four components. Only the first is always required, but you should know which ones you left out and why.

| Component | What it answers | Example |
| :--- | :--- | :--- |
| **1. Instruction** | What should the model do? | "Rewrite the instructions below in plain language." |
| **2. Context** (optional) | What background or perspective should it use? | "They are for my 82-year-old grandmother, who is not familiar with medical terms." |
| **3. Input Data** | What text, question, or file should it work on? | The pharmacy label text pasted below the request. |
| **4. Output Format** (optional) | What shape should the answer take? | "A numbered list of no more than five items, each under 12 words." |

Here is the complete prompt built from those four components:

```
Rewrite the pharmacy instructions below in plain language. They are for my 82-year-old grandmother, who is not familiar with medical terms. Present them as a numbered list of no more than five items, each under 12 words.

Instructions:
Take 1 tablet by mouth twice daily with food. Do not crush or chew. Avoid grapefruit juice while taking this medication. May cause drowsiness; use caution when driving or operating machinery. Complete the full course even if symptoms improve.
```

```{note}
The component that beginners omit most often is **Context**. Without it, the model writes for an average reader of an average request, which is rarely the reader you have in mind. One sentence about who the answer is for often changes the output more than any other addition.
```

### Exercise 1: Deconstruct This Prompt

Identify the four components in the prompt below.

```
You are an assistant coach for a youth soccer team. Using the attendance notes below, list the players who missed more than one practice this month. Present the answer as a two-column table with the headings Player and Practices Missed.

Notes:
Ana missed March 3 and March 10. Ben missed March 3. Carla missed March 3, March 10, and March 17. Diego attended every practice. Emma missed March 17.
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

The Atlantic hurricane season officially runs from June 1 to November 30. Most storms form between August and October, when ocean temperatures are highest, and the statistical peak of the season is around September 10. Forecasters issue a hurricane watch when hurricane conditions are possible within 48 hours and a hurricane warning when they are expected within 36 hours.
```

**2. Role-based prompt.** Assign the model a role or identity that shapes its tone and the knowledge it draws on.

```
You are a financial counselor at a university. A first-year student asks how to start building credit without getting into debt. Give practical advice in five bullet points.
```

**3. Chain-of-thought prompt.** Ask the model to work through the problem step by step before answering.

```
A family is driving 540 miles. Their car averages 30 miles per gallon, and gas costs $3.40 per gallon. They will split the fuel cost equally with another family. Work through the calculation step by step, then state how much each family pays.
```

The correct answer is $30.60 (18 gallons × $3.40 = $61.20, divided by 2). Showing the steps lets you check each one, which is the main practical benefit of this pattern.

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
A prompt can belong to more than one type at the same time. The instructional prompt above is also a zero-shot prompt, and the role-based prompt could become few-shot by adding examples of good advice.
```

```{note}
Many recent models reason step by step on their own before answering, so asking for chain-of-thought improves their answers less than it did with earlier models. It is still valuable for a different reason: it makes the reasoning **visible**, so you can check it.
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
   Explain what a credit score is to a first-year college student who has never had a credit card. Use exactly four bullet points, then end with one everyday analogy. Keep the whole answer under 150 words.
   ```

   Did the model follow every format instruction? Count the bullet points and the words yourself; do not assume.

3. **Add a role and chain-of-thought.** In a new conversation, paste:

   ```
   You are a financial counselor at a university. Explain what a credit score is to a first-year college student who has never had a credit card. First, list the factors that make up a credit score. Then explain which of those factors a student can control during the first year of college, and give one concrete action for each. Keep the whole answer under 200 words.
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
Summarize the library announcement below for a neighborhood newsletter. Use exactly three bullet points. Include the new hours, the laptop rules, and the cost of late returns. Use only the information in the announcement.

Announcement:
Starting March 1, the Riverside Public Library will extend its weekday hours to 9 a.m. to 8 p.m. Weekend hours remain 10 a.m. to 5 p.m. The library will also begin lending laptops to cardholders aged 18 and older. Laptops may be borrowed for up to 7 days and cannot be renewed. Late returns are charged $5 per day. The program is funded by a two-year state grant.
```

**Step 1, before running:** a good answer must contain (a) weekday hours, (b) weekend hours, (c) who can borrow laptops, (d) loan length and no renewal, (e) the $5 daily late fee, in three bullets.

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

The total is 18 out of 25, which sounds acceptable. It is not. A reader following this newsletter would arrive on a Saturday evening to a closed library, keep a laptop for two weeks, and be surprised by a $35 fine. **The accuracy gate fails, so the output fails.**

Notice also that the prompt already said "use only the information in the announcement". The model still added a claim. Instructions reduce errors; they do not eliminate them. That is why step 4 exists.

**Step 6, refining the prompt.** The weakest criterion is accuracy, and the errors cluster around hours and loan rules. One change:

```
Summarize the library announcement below for a neighborhood newsletter. Use exactly three bullet points: one for the hours (list weekday and weekend hours separately), one for the laptop rules (who, how long, renewals), and one for late returns. Use only the information in the announcement. Do not add benefits, prices, or rules that are not stated.

Announcement:
Starting March 1, the Riverside Public Library will extend its weekday hours to 9 a.m. to 8 p.m. Weekend hours remain 10 a.m. to 5 p.m. The library will also begin lending laptops to cardholders aged 18 and older. Laptops may be borrowed for up to 7 days and cannot be renewed. Late returns are charged $5 per day. The program is funded by a two-year state grant.
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

You can do it yourself, or ask the model to do it with a **meta-prompt** (a prompt about prompts):

```
Here is an AI-generated output:

1. Check whether your homeowner's or renter's insurance covers flood damage.
2. Photograph your belongings and store the photos online.
3. Pack a go-bag with medications, documents, water, and chargers.

Your task is to reverse prompt this output. Propose three different prompts that could have generated it. For each one, explain which words in the prompt would produce the numbered format, the practical tone, and the focus on preparation before a storm. Then explain how changing the audience in each prompt would change the output.
```

Compare the model's reconstructed prompts with your own guesses. Did either include a role? An audience? A number of items? Each feature you can explain is a feature you can now produce deliberately.

### 2.8 How Much Checking Is Enough?

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
You are a student-services advisor. Read the scholarship notice below and turn it into a checklist a student can follow to apply. Present a numbered checklist, then one line stating the deadline. Use only the information in the notice. If a student would need to know something that the notice does not state, list it under the heading "Not stated: ask the Fund".

Notice:
The Harbor Valley Community Fund Scholarship awards $2,500 to one student each year. Applicants must be enrolled full time at a public college or university in Florida and have a cumulative GPA of at least 3.0. Applicants must submit a 500-word essay describing a problem in their local community and how they would address it, plus two letters of recommendation, at least one of which must come from a faculty member. Applications are due November 15 at 5:00 p.m. Finalists will be interviewed in December, and the recipient will be announced on January 20.
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

1. **What you do not include, the model does not know.** If your budget is $60, and you do not say so, the model plans for an average budget.
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

**Connecting to HW7 (Prompt Iteration Log):** choose one real task from your own life, studies, or intended career. Write an initial prompt and then five successive revisions, changing one thing each time and running each in a new conversation. For each version, record what you changed and why, the five criterion scores, and any claims you checked. Your final version must use at least two context engineering techniques from Section 3.3; name them. End with the AI Disclosure statement required by the syllabus. The log is graded, not the final prompt.

## 📘 Further Reading

- Russell, S., & Norvig, P. (2022). _Artificial Intelligence: A Modern Approach_, 4th Ed., Ch. 24. Pearson.
- Mitchell, M. (2020). _Artificial Intelligence: A Guide for Thinking Humans_, Ch. 11–13. Penguin Books.
- Mollick, E., & Mollick, L. R. (2024). _Co-Intelligence: Living and Working with AI_. Portfolio.
- Prompt Engineering Guide. [https://www.promptingguide.ai/](https://www.promptingguide.ai/)
- White, J. et al. (2023). _A Prompt Pattern Catalog to Enhance Prompt Engineering with ChatGPT_. arXiv:2302.11382. [https://arxiv.org/abs/2302.11382](https://arxiv.org/abs/2302.11382)
- Wei, J. et al. (2022). _Chain-of-Thought Prompting Elicits Reasoning in Large Language Models_. arXiv:2201.11903. [https://arxiv.org/abs/2201.11903](https://arxiv.org/abs/2201.11903)
- Lewis, P. et al. (2020). _Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks_. arXiv:2005.11401. [https://arxiv.org/abs/2005.11401](https://arxiv.org/abs/2005.11401)
- Anthropic (2025). _Effective Context Engineering for AI Agents_. [https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- Dendritic Institute (2025). _AI Literacy Series — Module 5: Fundamentals of Prompt Engineering_ and _Module 6: Context Engineering_.



---

# Unit 7: Prompt Engineering

```
Sessions 11–12 | Sep 23, 28 | HW7 assigned Sep 28, due Oct 05
```

Part III begins here. For six units you have been studying the basic concepts of artificial intelligence and neural networks; from now on we are going to focus on generative systems. Most of what you build in Units 8 through 15 rests on the skills developed in this unit.

**A prompt is a specification.** You already know how to write specifications; you know that vague requirements produce wrong software, and that the fix is precision about inputs, outputs, constraints, and acceptance criteria. Prompting is the same discipline applied to a nondeterministic executor.

## Learning Objectives

After completing this unit, you will be able to:

- Decompose a prompt into its functional components and write each deliberately.
- Apply **role, few-shot, chain-of-thought, and output-format** techniques and say when each helps.
- Distinguish **prompt engineering** from **context engineering** and explain why the second matters more at scale.
- Design a **verification step** appropriate to the task's error cost.
- Build and maintain a reusable **prompt library**.

## Part I — Why Prompting Is Not Conversation

The failure mode for new users is treating the model as a colleague who shares your context. It does not. It has no access to your repository, your team's conventions, last week's meeting, or what you actually meant, unless you provide all this context to the model. 

### 1.1 Anatomy of an Effective Prompt

Six components. Not all are needed every time, but knowing which you omitted is important.

| Component | Question it answers | Example |
| :--- | :--- | :--- |
| **Role** | From what perspective? | "You are a senior backend engineer reviewing a pull request." |
| **Task** | What action, exactly? | "Identify race conditions in this function." |
| **Context** | What must it know? | "This runs in a multi-threaded request handler. `cache` is shared." |
| **Constraints** | What are the limits? | "Python 3.11, standard library only, no external dependencies." |
| **Format** | What shape is the output? | "A markdown table: line number, issue, severity, suggested fix." |
| **Criteria** | What makes it good? | "Flag only issues that could produce incorrect data, not style." |

```{note}
The component that beginners omit most often is **Criteria**, and it is the one that changes the output most. Without it, the model optimizes for what usually satisfies such a request, which is *thoroughness*. That is why unconstrained prompts return twelve suggestions when you wanted the two that matter.
```

### 1.2 Core Techniques

**Role prompting.** Assigning a perspective narrows the statistical territory the model draws from. "Explain OAuth" and "Explain OAuth to a backend developer who has implemented session auth but never a token flow" produce genuinely different text, because the second describes a much narrower region of the training distribution.

**Few-shot prompting.** Give two or three worked examples of input → output. This is by far the most reliable technique for enforcing a format, and it usually beats describing the format in words. If you need commit messages in a house style, showing three real ones works better than a paragraph of rules.

**Chain-of-thought.** Asking the model to work through steps before answering improves multi-step reasoning. The mechanism is worth understanding: the intermediate tokens become part of the context for the tokens that follow, so the model is conditioning on its own partial work. It is not "thinking harder", it is giving itself more relevant context.

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

This cell prints four variants of the same request, from bare to fully specified. Paste each into an AI assistant **in a separate conversation**, a fresh one each time, so earlier context does not leak, and score the results.

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

Unit 8 takes this further as workspaces and assistants are, mechanically, tools for controlling context deliberately instead of accidentally.

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
[PASTE CODE HERE]""",
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
[PASTE CODE HERE]""",
"verify": "Check the walkthrough against the code line by line. "
          "Verify every [INFERRED] claim separately."},

"debug_with_context": {
"prompt": """I have a bug I cannot locate. Do not guess at the cause.

First, list the five most likely causes given the symptoms, ordered by
probability, and for each one state the single cheapest test that would rule
it in or out.

Do not propose a fix until I tell you which test result I got.

SYMPTOM: [what you observe]
EXPECTED: [what should happen]
WHAT I HAVE ALREADY RULED OUT: [list, so it does not repeat your work]
RELEVANT CODE: [PASTE]
ERROR OUTPUT: [PASTE VERBATIM, DO NOT SUMMARIZE]""",
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
[PASTE]""",
"verify": "Check every DECISION against the thread - the most common error "
          "is promoting a proposal nobody objected to into a decision."},

"stress_test_my_reasoning": {
"prompt": """I have reached a conclusion and I want it attacked, not confirmed.

MY CONCLUSION: [state it plainly]
MY REASONING: [how you got there]
EVIDENCE I AM RELYING ON: [list]

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
- Anthropic (2025). _Prompt Engineering Overview._ [https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview]
- Dendritic Institute (2025). _AI Literacy Series — Module 2, Part VII: Your Prompt Library._
