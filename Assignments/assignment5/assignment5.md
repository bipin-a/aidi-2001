# Assignment 5: End-to-End AI Application

## Goal

In this assignment, you will:

* build a **small end-to-end AI-enabled application**
* move data through a realistic workflow from ingestion to user interface
* use required **skills** to support planning, implementation, testing, and architecture review
* deploy your product to **Vercel**
* demonstrate both your **working product** and your **development process**

The purpose of this assignment is to help you practice building a realistic proof of concept that includes data flow, architecture choices, testing, and deployment.

This is a **proof-of-concept assignment**, not a production engineering assignment. You are not being evaluated on scale. You are being evaluated on whether you can build and explain a focused, working system.

---

# What You Must Submit

Submit **all 3** of the following:

## 1. A video walkthrough

Record a **10-minute demo video** with:

* **screen recording**
* **audio**
* **webcam on**

Your video must walk me through:

* your product
* your architecture
* your development workflow
* your deployed app
* your repository artifacts

## 2. A deployed application URL

You must deploy your application on **Vercel** and submit the live URL.

* do **not** submit a localhost URL
* your submitted app must be accessible through a real deployed link

## 3. A public GitHub repository URL

Your repository must be **public** by the submission deadline.

I need to be able to inspect:

* your code
* your GitHub Issues
* your commit history
* your tests
* your workflow artifacts

---

# Assignment Overview

You will complete **3 parts**:

1. **Plan the Product**
2. **Build the System**
3. **Deploy and Demonstrate**

---

# Part 1: Plan the Product

## Project requirement

Build an application of your choice, but it must be:

* small enough to complete for this assignment
* narrow enough to be reliable
* complete enough to demonstrate an end-to-end workflow

The goal is **not** to build the biggest app possible. The goal is to build a **focused, working, well-explained proof of concept**.

## Your application must cut across these layers

Your product must move through multiple layers of a software system, including:

* **raw data / source ingestion**
* **ETL / transformation**
* **storage**
* **LLM or reasoning layer**
* **UI / user-facing application**

Your project should not just be a front end, and it should not just be a single LLM prompt. It should show how data moves through a system into a usable application.

## Supported use case

Your app must support a clear set of user tasks or questions.

You must define:

* what your app does
* what questions or tasks it supports
* what is out of scope

A strong project is one that does a few things clearly and reliably.

---

# Part 2: Build the System

## Required workflow skills

You must install and use these 5 skills from Matt Pocock's collection:

<https://github.com/mattpocock/skills/tree/main>

* `grill-me`
* `write-a-prd`
* `prd-to-issues`
* `tdd`
* `improve-codebase-architecture`

You are strongly encouraged to use these skills in **Codex** as part of your real workflow for this assignment. The goal is not just to install them, but to use them intentionally while planning, building, testing, and refining your project.

## Helpful links

If you want a practical overview of how these 5 skills are commonly used together, see:

* Matt Pocock, "5 Agent Skills I Use Every Day"  
  <https://www.aihero.dev/5-agent-skills-i-use-every-day>

## Windows setup for `npx`

If you are using Windows and do not already have `npx`, install **Node.js LTS** first. `npx` comes with `npm`, and `npm` comes with Node.js.

### Option 1: install with `winget`

Open **PowerShell** or **Command Prompt** and run:

```powershell
winget install OpenJS.NodeJS.LTS
```

Then close and reopen your terminal and verify:

```powershell
node -v
npm -v
npx -v
```

### If `npx` is still not found

In most cases, reopening the terminal is enough. If it still does not work, check that the Node.js install directory is on your `PATH`.

Typical Windows path:

```text
C:\Program Files\nodejs\
```

After updating `PATH`, open a new terminal and run:

```powershell
node -v
npm -v
npx -v
```

## Required workflow order

Use the skills in this order:

1. `grill-me`
2. `write-a-prd`
3. `prd-to-issues`
4. `tdd`
5. `improve-codebase-architecture`

Do not treat these as disconnected checkboxes. They are meant to support a realistic workflow from planning to implementation to refinement.

## What these skills should help you do

* `grill-me` -> challenge and sharpen your plan before building
* `write-a-prd` -> define what you are building and produce a parent PRD GitHub Issue
* `prd-to-issues` -> break the PRD into smaller GitHub Issues
* `tdd` -> implement meaningful behavior with tests
* `improve-codebase-architecture` -> review and improve structure after the first version works

## Minimum acceptable evidence for each skill

### `grill-me`

You must show evidence that you used `grill-me` to pressure-test your idea.

Minimum evidence:

* a saved chat excerpt, screenshot, or notes from the interaction
* a short explanation of what changed because of the grilling

### `write-a-prd`

You must create at least one **parent PRD GitHub Issue**.

Minimum evidence:

* a link to the PRD issue
* clear scope, use case, architecture direction, and out-of-scope notes

Important:

* use `write-a-prd` to produce the PRD and file it as a parent GitHub Issue
* you may create **multiple PRDs**
* multiple smaller PRDs are often smarter than one giant PRD

### `prd-to-issues`

You must turn the PRD into actionable GitHub Issues.

Minimum evidence:

* linked GitHub Issues created from the PRD
* issues that represent smaller vertical slices of work
* issues that are more specific than "build the whole app"

### `tdd`

You must include tests in your repo, and those tests must make sense for the behaviors your app claims to support.

Minimum evidence:

* test files in the repo
* tests tied to real supported features or behaviors
* tests that are understandable and relevant, not random or superficial

I will be looking at whether the tests make sense.

### `improve-codebase-architecture`

You must show evidence that you reviewed and improved the codebase structure after an initial version worked.

Minimum evidence:

* a short before/after architecture note, or
* a linked architecture/refactor issue, plus
* evidence in commits or code structure that meaningful improvement happened

## Testing requirements

You must satisfy both of the following:

* use `tdd` for meaningful parts of your implementation
* use **Playwright MCP** for at least some meaningful UI/UX or end-to-end test scenarios

Your submission must include evidence of both:

* tests for core logic or supported behaviors
* browser-based testing that exercises the real user experience

## Minimum acceptable evidence for Playwright MCP

* evidence that Playwright MCP was used during development
* at least one meaningful UI or end-to-end test scenario
* testing that reflects real user behavior, not just trivial page-load checks

## Recommended architecture

The **recommended default architecture** for this assignment is:

* ingest raw data
* run ETL using **GitHub Actions**
* store raw and processed data in object storage
* use curated files as the app's source of truth
* deploy the app on **Vercel**

This is a strong architecture for this assignment because it is:

* realistic
* simple enough to complete
* repeatable
* easy to explain
* appropriate for a POC

## Allowed storage and infrastructure choices

* **GCP buckets** are recommended because they are simple for this assignment and offer student-friendly free-tier credits
* **AWS S3** or another equivalent object storage solution is also acceptable if used sensibly
* tools such as Supabase, Neon, Pinecone, OCR tools, scheduled jobs, APIs, or other managed services are allowed when they clearly support your use case
* choose the **simplest** storage and infrastructure approach that supports your proof of concept

## Storage guidance

For this assignment, you do **not** need to build a full vector store or feature store unless your use case clearly requires it.

In many cases, a POC can work well with:

* JSON files
* CSV files
* markdown files
* other curated files stored in object storage

That is acceptable for this assignment.

Your goal is to prove the workflow, not to build infrastructure for scale.

## Simplicity expectation

Keep this project simple.

This means:

* choose the simplest architecture that proves the idea
* do not add tools just because they exist
* do not overbuild storage or infrastructure
* do not spend most of your time chasing platform complexity

Use managed tools only where they clearly help the project.

## Important development advice

Do **not** build everything at once.

A strong workflow is to test simple pieces early:

* confirm keys and environment variables work
* confirm GitHub Actions runs successfully
* confirm ETL can move data into storage
* confirm the app can read from curated files
* confirm deployment works
* then connect the system piece by piece

It is much better to prove one narrow flow works end to end than to partially build five disconnected parts.

## Multiple PRDs are encouraged

If you are smart about scope, you may use multiple PRDs.

For example:

* PRD 1: ingest raw data and write cleaned output to storage
* PRD 2: connect the app to curated files
* PRD 3: add the LLM or reasoning layer
* PRD 4: improve testing and architecture

This is often better than trying to design and build the entire system in one giant plan.

## Data expectations

Your app should ingest real or realistic source data, such as:

* APIs or JSON
* PDFs
* websites
* documents
* semi-structured or unstructured content

## ETL expectations

Your project must include a repeatable transformation step that turns raw input into usable application data.

Examples include:

* extraction
* cleaning
* joining
* chunking
* OCR
* metadata generation
* schema normalization
* scheduled or automated processing

For most students, **GitHub Actions** is the preferred way to run this ETL.

## Storage expectations

You must store your data in a meaningful way.

A staged structure is recommended, such as:

* **Bronze** -> raw source files
* **Silver** -> cleaned or enriched files
* **Gold** -> final curated files used by the app

For this assignment, these may simply be files in object storage such as **GCP buckets** or **AWS S3**.

That is enough for many strong POCs.

## AI / reasoning expectations

Your project must include an LLM-enabled or reasoning-enabled layer.

You may use:

* **RAG**
* **tool calling**
* **a hybrid approach**

Choose the approach intentionally.

Examples:

* use **RAG** when retrieval and summarization are enough
* use **tool calling** when you need more deterministic answers
* use **a hybrid approach** when both are useful

Your architecture should match the kind of questions or tasks your app supports.

---

# Part 3: Deploy and Demonstrate

## Deployment requirement

You must deploy your app on **Vercel**.

Your submission must include:

* a working **Vercel URL**
* a **public GitHub repository URL**

## What your repository must show

Your repo must include:

* the project code
* a clear README
* GitHub Issues created from your PRD workflow
* commit history showing your development process
* tests
* evidence of architecture improvements

## What you must show in the video

Your **10-minute video** must show all of the following:

* your final working deployed product
* the live **Vercel URL**
* your public GitHub repository
* your GitHub Issues
* where and how you used each required skill
* your architecture at a high level
* the data flow through your system
* what your app can do
* what your app does **not** support
* at least one meaningful end-to-end walkthrough of the product
* evidence that Playwright MCP was used for some testing

---

# Examples That Do Not Meet Requirements

The following do **not** meet the assignment requirements on their own:

* a simple chat UI over one prompt with no ingestion, ETL, or storage
* a front end that only works with localhost and is not deployed
* a repo with no meaningful Issues, tests, or commit history
* an app that gestures at architecture but does not actually move data through a pipeline
* overengineered infrastructure with no clear POC value

---

# Scope Examples

## Too small

* a one-page chat app that sends one hardcoded prompt to an LLM
* a UI mockup with no real data flow
* a local script with no deployment and no user-facing product

## Acceptable

* ingest PDFs or JSON, process them with GitHub Actions, store curated files in object storage, and expose a narrow Q&A or dashboard interface on Vercel
* ingest website data, clean and organize it into files in storage, and build a small app that supports a few well-defined user questions
* build a focused document analysis or lookup tool with a limited, clear feature set

## Too large

* a multi-tenant SaaS platform
* a product that tries to support many unrelated workflows
* a system with unnecessary databases, vector infra, auth layers, schedulers, and integrations before the core POC works

---

# Suggested Project Ideas

Choose something appropriately scoped, such as:

* a document Q&A tool over a specific dataset
* a PDF analysis app
* a small dashboard answering a few defined questions
* a structured lookup tool with AI explanations
* a narrow research assistant over curated sources
* a search and summarization interface over processed documents

---

# Video Requirements

## Format

* **10 minutes maximum**
* **screen share required**
* **audio required**
* **webcam on**

## Your video must cover

* the product and its use case
* the deployed Vercel app
* the repo and commit history
* the GitHub Issues
* the required skills and where they were used
* the system architecture
* the data flow through the system
* a live walkthrough of the product
* the tests, including Playwright MCP usage

---

# Required Checklist

Use this checklist before submitting.

## Planning

* [ ] I chose a project that is small enough to complete
* [ ] I clearly defined what my app does
* [ ] I used `grill-me`
* [ ] I created at least one PRD issue with `write-a-prd`
* [ ] I created GitHub Issues from `prd-to-issues`

## Build

* [ ] My project includes ingestion
* [ ] My project includes ETL or transformation
* [ ] My project includes storage
* [ ] My project includes an LLM or reasoning layer
* [ ] My project includes a user-facing UI
* [ ] I used `tdd`
* [ ] My tests make sense for my supported features
* [ ] I used Playwright MCP for meaningful UI/UX or end-to-end tests
* [ ] I used `improve-codebase-architecture`

## Architecture

* [ ] My system is a simple POC, not an overbuilt production design
* [ ] I used simple storage where appropriate
* [ ] I avoided unnecessary infrastructure
* [ ] I can explain why I chose my architecture

## Deployment

* [ ] I deployed my project on Vercel
* [ ] I tested the deployed application
* [ ] I am submitting a real deployed URL, not localhost

## Submission

* [ ] My GitHub repository is public
* [ ] I recorded a 10-minute video with screen share, audio, and webcam
* [ ] I showed the final product in the video
* [ ] I showed my repo in the video
* [ ] I showed my GitHub Issues in the video
* [ ] I showed where I used each required skill
* [ ] I submitted my Vercel URL
* [ ] I submitted my public GitHub repository URL

---

# Final Summary

## You are required to:

* build an **end-to-end AI-enabled application**
* move through **ingestion, ETL, storage, reasoning, and UI**
* use all **5 required skills in order**
* show evidence of your workflow through **PRDs, Issues, tests, commits, and architecture review**
* use **Playwright MCP** for some meaningful UI/UX or end-to-end testing
* deploy the app on **Vercel**
* use a **public GitHub repository**
* submit a **10-minute video demo**
* submit your **Vercel URL**
* submit your **public GitHub repository URL**
