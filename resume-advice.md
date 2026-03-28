# AI-Assisted Resume Tailoring Process

I use AI to tailor my resume to a job, not to invent experience or rewrite everything blindly.

The goal is to make the resume:

* relevant to the role
* easy for ATS systems to parse
* clear to a human reader

Start with an ATS-friendly resume and put your **resume + full job description** in the same chat. If you have multiple resume versions, include those too.

---

## 1. Compare resume to job description

First, identify the gaps and choose the best starting point.

**Prompt:**

> Compare my resume to this job description. Give me:
>
> 1. a match score out of 100
> 2. the top missing keywords, skills, or qualifications
> 3. the weakest bullets or sections
> 4. any experience that should be emphasized or reframed for this role
> 5. if I uploaded multiple resumes, tell me which one is the best base and what I should pull from the others

**Why:**
This tells you what actually needs to change before you start editing.

**HITL note:**
Do not treat the score as truth. Use it as a diagnostic. Check whether the “missing” items are things you actually have experience with, or just things the model wants because they appear in the job description.

---

## 2. Surface missing evidence

Check whether relevant work is missing entirely.

**Prompt:**

> Based on the job description and the gaps you identified, what kinds of projects, results, tools, or experiences seem missing from my resume but may exist in my background and be worth adding? Only suggest things I may plausibly already have done based on my existing experience. Do not invent experience.

**Why:**
Sometimes your best evidence is missing, not just poorly worded.

**HITL note:**
Pause here and review your actual experience. Ask yourself: *Is there more?* Think about projects, ownership, tools, results, side work, internal initiatives, cross-functional work, or impact you have not written down yet. This is the point where you fill in gaps from memory, not where you let the model guess.

---

## 3. Rewrite the resume

Now rewrite using the findings from the first two steps.

**Prompt:**

> Rewrite my resume for this role based on the job description and the gaps we identified. Keep everything truthful, specific, and results-oriented. Improve weak bullets, naturally incorporate relevant keywords, and prioritize outcomes over responsibilities. Do not fabricate metrics, tools, or scope. If stronger metrics would help, flag where they are missing.

**Why:**
This turns the analysis into an actual tailored draft.

**HITL note:**
Read every rewritten bullet and ask: *Would I say this out loud in an interview?* If not, rewrite it. Do not keep bullets that sound too polished, too vague, or unlike how you actually worked.

---

## 4. Cut noise and combine bullets

Remove or merge anything that is not helping.

**Prompt:**

> Review this revised resume for signal-to-noise ratio against the job description. Tell me which bullets or sections should be cut, shortened, or combined because they are not among the strongest evidence for this role.

**Why:**
A stronger resume is usually the one with less irrelevant detail.

**HITL note:**
Be willing to cut bullets that are good but not useful for this role. More content is not better. Ask yourself whether each line earns its space.

---

## 5. Check ATS structure

Make sure the format is easy to parse.

**Prompt:**

> Review this resume like an ATS parser. Flag any structural or formatting issues that could make it harder to parse, including headings, section names, bullet formatting, dates, links, or layout choices.

**Why:**
A good resume can still underperform if the structure is messy.

**HITL note:**
Do not rely on the model alone here. Also manually scan the document formatting yourself. Keep it simple, standard, and readable.

---

## 6. Validate in a fresh chat

Do a final cold read.

**Prompt:**

> Act as a recruiter reviewing this resume for the first time. Score it against the job description out of 100 and briefly explain what is strong, what is weak, and what still feels missing or overdone.

**Why:**
A fresh chat gives a better test than the same one that helped edit it.

**HITL note:**
Use this as a final check, not a final authority. If the resume scores well but no longer sounds like you, fix that before sending it.

---

## In short

**Compare → Surface missing evidence → Rewrite → Cut noise → ATS check → Validate**

---

## Key point

This should not be an end-to-end automated process.

AI helps you spot gaps, improve wording, and tighten relevance. You still need to decide what is true, what is important, and what best represents your experience.

