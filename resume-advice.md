# How to Use AI to Improve Your Resume in 2 Phases

This is glimpse into my current resume creation/tailoring strategy.

Most people throw their resume at an LLM and ask it to "make it better." That produces generic fluff. This guide gives you a repeatable loop: first make the content strong, then make it fit on one page — without gutting what matters.

---

## Before You Start

Grab a clean, ATS-friendly base template. Fancy columns, icons, and graphics get mangled by applicant tracking systems. Start simple.

The template that I use: [SheetsResume](https://sheetsresume.com/resume-template)

Attach your current resume document **and** the full job description into a new chat. Both need to be in context for every prompt below.

---

## Phase 1: Improve Relevance and ATS Match

In this phase, you're making your resume clearly aligned to a specific job description.

**A tip:** I'll often run the same prompts in both Claude and ChatGPT, then cherry-pick the best bullets from each. Similar to the idea behind [llm-council](https://github.com/karpathy/llm-council) — I'll copy results from the LLM I like better and ask the other one to review, flag disagreements, and suggest improvements.

### Prompt 1 — Gap Analysis

> Act as a senior recruiter. Compare my resume against this job description. Give me:
> 1. A match score out of 100
> 2. The top 5 missing keywords, skills, or qualifications
> 3. The weakest bullets or sections
> 4. Any experience that should be reframed for this role

This gives you a baseline score and a clear hit list of what's missing.

### Prompt 2 — Rewrite for Impact

> Rewrite my experience section to naturally incorporate those missing keywords. Use the Google XYZ formula for every bullet. Keep it truthful, specific, and results-oriented. Do not fabricate metrics, tools, or scope — if I haven't provided a number, flag it and ask me. Use strong action words to start off each bullet.

The Google XYZ formula means: "Accomplished [X] as measured by [Y] by doing [Z]." The point is that every bullet should show what you did, how it was measured, and what action drove it.

### Prompt 3 — ATS Readability Check

> Act as an applicant tracking system parser. Scan my updated resume and flag any sections, formatting choices, or wording that an ATS bot would struggle to parse or classify correctly.

This catches things like unrecognized headers, date format issues, acronyms without spelled-out versions, or skills buried inside prose instead of being scannable.

---

## Phase 2: Fit It to One Page

Stay in the same chat from Phase 1 — the LLM needs the full context of what you've built so far. I've found this phase works best with Claude.

Now that the content is strong, **compress**. The guiding principle:

**Cut anything that duplicates a signal already shown elsewhere, carries no metric or concrete outcome, or is less relevant to the target role.**

You're not making the resume shorter at random. You're making every line earn its place.

### Step 1 — Cut Content First

> My resume is currently over one page. Help me cut it down. Apply these rules in order:
> 1. Remove any bullet that duplicates a point made elsewhere or lacks a measurable outcome.
> 2. Drop experiences or sections least relevant to this specific job description.
> 3. Only after cutting, condense remaining bullets for brevity without losing the metric or the action.
>
> Show me what you cut and why.

Content cuts come first. Shrinking margins and font size to cram in weak bullets defeats the purpose.

### Step 2 — Then Tighten Spacing

> Now tighten the layout for a one-page resume. Reduce section heading spacing, bullet spacing, and contact section whitespace. Preserve readability — only compress where it doesn't hurt scannability.

Spacing adjustments are the finishing pass, not the first move. You can keep asking the LLM to reduce spacing incrementally. For reference, I have mine set to: section heading spacing 160/40, bullet spacing 20/20, contact gap 80.

**Note**: Sometimes after this tightening step I'll notice I have enough white space to add more things which I removed previously! 


---

## The Iteration Loop

Don't stop after one pass. Go back and forth between Phase 1 and Phase 2 until the resume is both strong for the role and clean enough for one page.

Then do a final validation:

1. **Open a brand-new chat** (Claude or ChatGPT — either works). A fresh chat has no memory of your editing session, so it judges the resume cold.
2. Attach your latest resume and the job description.
3. Use: *"Act as a senior recruiter. Score this resume against the job description out of 100 and explain the score briefly."*
4. Check the score against the table below.
5. Repeat until you land in the sweet spot.

---

## Scoring Guide

| Score | What It Means | Action |
|-------|--------------|--------|
| Below 70 | Major gaps in keywords or relevance | Heavy Phase 1 work needed — you may be missing real experience, not just wording. Consider building new projects. |
| 70–79 | Solid foundation, real gaps remain | Targeted Phase 1 tweaks, then Phase 2 |
| 80–84 | Strong — review what's missing, decide if it matters | Minor tweak required |
| **85–90** | **Sweet spot — clearly tailored, still sounds human** | **Nice** |
| Above 90 | Likely over-fitted to the job description | Read it out loud — if it sounds like a job description instead of a person, dial it back |

Keep in mind these scores come from an LLM role-playing as a recruiter, not a standardized system. Don't treat 84 vs 86 as meaningfully different. The point is to give yourself a rough compass, not a precise measurement.

---

## Final Notes

This process will not create new experience. If you're lacking experience, you need to go build presentable things — no amount of prompt engineering fixes an empty resume.

The good news: once you've done this for one role, it gets much faster for similar ones. A data scientist posting at Company A and Company B will overlap heavily, and you won't need to redo the full Phase 1–2 loop each time.

*The goal isn't to trick an ATS. It's to make sure the system doesn't throw away a resume that a human would want to read.*