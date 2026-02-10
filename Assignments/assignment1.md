# Assignment 1 — Investigating Transformer Models

**Course: Deep Learning for Natural Language Processing**

---

## How This Assignment Works

You are not following a tutorial.

You are studying a system.

Your goal is to behave like a scientist:

> observe → experiment → compare → explain

You will design measurements, choose visualizations, and justify conclusions.

There is no single correct output — grading is based on reasoning quality.

---

## Expected Effort

Expected time: **10–15 hours**

For each section:

* 3–5 experiments
* 1 visualization or table
* 1 short explanation (1–2 sentences)

More experiments without new insight **will not** increase your grade.

You are finished when your explanation predicts what the model will do before running it.

---

## Learning Goals

By completing this assignment you should be able to:

* Interact with pretrained transformer models
* Study how language is represented inside NLP systems
* Analyze generation behavior
* Compare BERT and GPT behavior
* Understand why tokenizer and model must match
* Apply language models to structured data

You are **not expected to understand internal mathematics yet**.

---

## Environment Setup

Install:

* transformers
* datasets
* torch
* seaborn
* pandas
* matplotlib

Documentation:

* [https://huggingface.co/docs/transformers](https://huggingface.co/docs/transformers)
* [https://huggingface.co/learn/llm-course](https://huggingface.co/learn/llm-course)

---

# Part 1 — How Models Read Text

Transformers do not read words directly.
They convert text into another representation before processing.

Use the **BERT tokenizer** (`bert-base-uncased`) to investigate.

---

## Investigation

Create sentences varying in:

* capitalization
* punctuation
* numbers
* rare words
* length

Observe how they are represented.

Then load the seaborn **tips dataset** and convert rows into natural language descriptions.
Study how representation length varies across rows.

### Hints

* Try changing only one property at a time
* Character length and token length are different
* Numbers often behave differently than words

---

## Required Evidence

Include:

* One table or figure comparing at least 5 different sentences
* One observation about numbers or punctuation
* One observation about length differences
* One explanation of why token count varies

---

# Part 2 — Generating Language (GPT)

Language models generate text by repeatedly predicting tokens.

Use a GPT-style model.

Use at least these prompts:

* "The capital of France is"
* "I opened the fridge and found"
* "The number 1024 is important because"
* A description derived from the seaborn dataset

---

## Investigation

Generate multiple outputs from the same prompt under different generation settings.

### Hints

* Run the exact same prompt multiple times
* Compare stability vs creativity
* If outputs differ, decide whether randomness or rules caused it

---

## Required Evidence

Include:

* Outputs from the same prompt generated at least 3 times
* A comparison between two generation settings
* One concrete example of repetition or instability
* A short explanation of what controls creativity

---

# Part 3 — When Models Refuse to Behave (BERT vs GPT)

Some transformer models are not designed to generate text.

Test:

> "Deep learning models are very"

Compare BERT and GPT using multiple approaches.

### Hints

* Ask whether the model can see future words
* Try inserting a blank in the sentence
* A model can guess missing words but fail at continuing stories

---

## Required Evidence

Include:

* One example where GPT succeeds
* One example where BERT fails
* One example where BERT works differently than GPT
* A behavioral explanation of what each model is predicting

---

# Part 3.5 — Can We Swap Their Brains? (Tokenizer vs Model)

Are transformer components interchangeable?

---

## Investigation

Try mismatched combinations:

* GPT model with BERT tokenizer
* BERT model with GPT tokenizer

Observe outputs and errors.

### Hints

* The model only sees token IDs
* Consider what happens if an ID represents a different word than during training

---

## Required Evidence

Include:

* Evidence from at least one mismatched pair
* A description of what changed in the output
* A hypothesis about what the model expected but did not receive

### Final Question

Explain:

> Why a pretrained model generally cannot be paired with a different tokenizer.

Explain what specifically breaks.

---

# Part 4 — Compressing Information (Summarization)

Some transformer models rewrite long text into shorter text.

---

## Investigation

Create three inputs:

1. Natural paragraph
2. Loaded the penguins dataset from seaborn.
3. Mixed structured + natural text

Observe how summaries differ.

### Hints

* The model may preserve meaning even if details change
* Consistent mistakes matter more than rare ones

---

## Required Evidence

Include:

* One summary that works well
* One summary that loses important information
* One consistent mistake
* A short explanation of why structured data is difficult

---

# Submission

Submit a Jupyter Notebook containing:

* Experiments
* Visualizations or tables
* Short explanations
