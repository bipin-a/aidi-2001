# Assignment 6: Evaluate, Defend, and Improve Your AI Application

## At a glance

In Assignment 6, you will build on your **Assignment 5** app.

You will:

* explain your architecture
* evaluate your system
* compare against a simple baseline
* identify a failure point
* improve the system based on evidence

You are **not** building a brand-new app. You are improving and defending the one you already built.

---

# What You Must Submit

Submit all 3 of the following:

## 1) Video walkthrough

Record a **12-minute maximum** video with:

* screen recording
* audio
* webcam on

## Important video note

Your video should sound like a natural explanation of your own work, not a script read aloud. You may pause, stumble, restart a sentence, or speak casually. You are **not** being graded on polished English, accent, or presentation style. You **are** being graded on whether you clearly understand and can explain your own system.

A video that sounds heavily scripted or read word-for-word from notes will receive a **significant deduction on the video component**, because it makes it difficult to judge your actual understanding of the project.

## 2) Deployed application URL

Your app must still be deployed on **Vercel**.

## 3) Public GitHub repository URL

You must continue from your **Assignment 5 repository**.

Your repository must be public by the deadline.

---

# What I Will Look For

I should be able to inspect:

* your deployed app
* your code
* your tests
* your evaluation artifacts
* your commit history
* your evidence of improvement

---

# Assignment Overview

You will complete 4 parts:

1. **Classify and justify your architecture**
2. **Show your pipeline and data flow**
3. **Evaluate the system**
4. **Improve the system based on evidence**

---

# Part 1: Classify and Justify Your Architecture

## Choose one architecture category

Classify your system as:

### Prompt-first / long-context

Your system mainly relies on placing relevant information directly into the model context.

### Retrieval-first / RAG

Your system mainly retrieves stored content and passes retrieved context to the model.

### Tool-first / function-calling

Your system mainly relies on tool calls, functions, or other deterministic actions coordinated by the model.

### Hybrid

Your system meaningfully combines two or more of the above.

---

## In your video, explain

* what your app does
* what user tasks or questions it supports
* what is out of scope
* which architecture category best fits your system
* why you chose it
* what the main alternative was
* why you did not choose that alternative

You must discuss at least these tradeoffs:

* amount of data / number of files
* context-window limits
* retrieval or storage needs
* determinism needs
* cost
* operational overhead
* performance
* ease of debugging

### Also required

You must discuss **one important architecture capability you did not implement**, such as **RAG** or **tool calling**.

Explain:

* whether it would improve your system
* what problem it would solve
* what new complexity it would introduce
* under what future conditions you would adopt it

---

# Part 2: Show Your Pipeline and Data Flow

## What your repository must show

Your repo must make it clear:

* what your raw input is
* how it is transformed
* what gets stored
* what the app uses as its source of truth
* where errors could happen

This can be shown through existing repo materials such as:

* your README
* diagrams
* code structure
* config files
* test fixtures
* evaluation scripts
* logged outputs

You do **not** need a separate written report.

---

## In your video, walk through

* raw input
* preprocessing or transformation
* stored artifacts
* prompt construction, retrieval, or tool use
* model output
* UI display

Also explain the internal information your system keeps, or should keep, for debugging and evaluation, such as:

* prompt text
* model name or version
* retrieved chunks, if applicable
* tool calls, if applicable
* transformed records
* output examples
* run metadata

Keep this practical. It does not need to be overbuilt.

---

# Part 3: Evaluate the System

## Everyone must evaluate these 3 things

### 1) Output quality

Evaluate whether the app’s main output is good enough for its intended task.

Examples:

* answer quality
* summary quality
* extraction quality
* recommendation quality
* explanation quality

### 2) End-to-end task success

Show whether the full system completes representative user tasks from start to finish.

### 3) One upstream component

Evaluate one important upstream part of your pipeline before the final output.

Examples:

* parsing
* extraction
* chunking
* schema normalization
* file transformation
* metadata generation
* retrieval
* tool routing

---

## Only include these if they apply

### Retrieval evaluation

Required only if your app uses retrieval or RAG.

Examples:

* hit rate
* recall@k
* whether the correct document or chunk appears

### Tool / routing evaluation

Required only if your app uses tool calling, functions, or multi-step routing.

Examples:

* whether the correct tool was selected
* whether the tool output was correct
* whether routing was reliable

### Consistency / reproducibility evaluation

Required only if consistency is an important claim of your system.

Examples:

* repeated runs on the same input
* paraphrased prompts
* stability of retrieval or tool behavior

If you include this, define what “acceptable consistency” means for your app.

---

## Minimum evaluation requirement

You must include at least:

* **5 representative cases**
* **2 failure cases**
* **1 lightweight baseline comparison**

---

## What counts as a lightweight baseline

Your baseline does **not** need to be a second full system.

Acceptable examples include:

* a naive prompt-only version
* a no-retrieval version
* a no-tool version
* an earlier simpler prompt
* an earlier chunking strategy
* an earlier preprocessing method
* a manually simplified architecture choice

The goal is to show that your final design was not arbitrary.

---

## Metrics guidance

Choose metrics that match your task.

Examples:

* summarization → ROUGE, semantic similarity, or rubric scoring
* question answering → exact match, F1, BLEU, or semantic similarity
* extraction → field accuracy, precision, recall, F1
* retrieval → hit rate, recall@k, precision@k
* routing → routing accuracy or tool correctness

You do not need to use the same metric everywhere.

You do need to explain why your metric makes sense.

---

# Part 4: Improve the System Based on Evidence

## Required improvement

Based on your evaluation, make **at least one meaningful improvement** to your system.

Examples:

* improve chunking
* improve parsing or transformation
* refine prompts
* simplify an overcomplicated architecture
* add retrieval
* remove retrieval
* improve tool routing
* improve data logging for debugging
* improve the UI after end-to-end testing

---

## In your video, explain

* what looked weak or failed
* what evidence showed that
* what you changed
* what improved
* what still remains weak

The improvement does not need to make the system perfect. It does need to be real, motivated, and supported by evidence.

---

# Project Continuity Requirement

This assignment builds directly on **Assignment 5**.

You may refactor, simplify, extend, or partially redesign the system, but it must still clearly be an **evolution of the same application**, not a completely unrelated new project.

---

# What Your Repository Must Show

Your repo must include:

* the updated working application
* a clear README
* tests
* evaluation scripts, notebooks, or similar artifacts
* saved evaluation cases or fixtures
* results or outputs from those evaluations
* evidence of at least one improvement based on evaluation

Simple artifacts are fine here. Screenshots, saved outputs, small scripts, notebooks, and test fixtures are all acceptable if they clearly support your claims.

---

# What You Must Show in the Video

Your 12-minute video must show:

* your deployed Vercel app
* your public GitHub repository
* your architecture category
* why you chose it
* the main alternative you rejected
* one important capability you did **not** implement, and when you would add it
* your data flow through the system
* one upstream component evaluation
* your output-quality evaluation
* your end-to-end evaluation
* at least one failure case
* your lightweight baseline comparison
* at least one improvement you made because of evaluation
* what the app supports
* what the app does not support

Explain the walkthrough in your own words as the person who built the system. You are not expected to sound polished or scripted.

---

# Examples That Do Not Meet Requirements

These do **not** meet Assignment 6 requirements on their own:

* re-submitting Assignment 5 with no new evaluation
* saying “I used RAG” or “I used tools” without justification
* including tests but not explaining what they show
* discussing architecture without cost, overhead, and performance tradeoffs
* showing only a final demo with no failure analysis
* building a completely unrelated new app instead of improving the Assignment 5 one

---

# Required Checklist

## Continuity

* [ ] I extended my Assignment 5 project rather than starting an unrelated new one
* [ ] My deployed app is still live on Vercel
* [ ] My repository is public

## Architecture

* [ ] I classified my system as prompt-first, retrieval-first, tool-first, or hybrid
* [ ] I explained why I chose that architecture
* [ ] I explained the main alternative I did not choose
* [ ] I explained one important architecture capability I did not implement
* [ ] I explained when I would add it and why
* [ ] I discussed cost, overhead, and performance tradeoffs

## Pipeline / Data Flow

* [ ] I showed the stages of my system
* [ ] I showed what data moves between stages
* [ ] I included enough internal information to support debugging and evaluation

## Evaluation

* [ ] I evaluated output quality
* [ ] I evaluated end-to-end task success
* [ ] I evaluated one upstream component
* [ ] I included at least 5 representative cases
* [ ] I included at least 2 failure cases
* [ ] I compared against a lightweight baseline
* [ ] I justified my metrics

## Conditional Evaluation

* [ ] I evaluated retrieval if my system uses retrieval
* [ ] I evaluated tool or routing behavior if my system uses tools or routing
* [ ] I evaluated consistency if consistency is an important claim of my system

## Improvement

* [ ] I made at least one meaningful improvement based on evidence
* [ ] I showed what changed
* [ ] I showed what improved

## Submission

* [ ] My repo contains the updated project
* [ ] My repo contains evaluation artifacts
* [ ] My video explains my architecture, evaluation, failure cases, and improvement
* [ ] My video sounds like a natural explanation of my own work, not a script read aloud
* [ ] My video shows that I understand and can explain my system clearly
* [ ] I submitted my Vercel URL
* [ ] I submitted my public GitHub repository URL

---

# Final Summary

You are required to:

* extend your **Assignment 5** AI application
* keep it deployed on **Vercel**
* classify your system as **prompt-first, retrieval-first, tool-first, or hybrid**
* justify that architecture
* explain the main alternative you did not choose
* explain one important capability you did **not** implement and when you would add it
* evaluate **output quality**, **end-to-end success**, and **one upstream component**
* include retrieval, tool/routing, and consistency evaluation **only if applicable**
* include **5 representative cases**, **2 failure cases**, and **1 lightweight baseline**
* make at least one evidence-based improvement
* submit your **video**, **Vercel URL**, and **public GitHub repository URL**
