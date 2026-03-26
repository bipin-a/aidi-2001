# How to Use AI to Tailor Your Resume — A Repeatable Process

This is a look into my current resume tailoring strategy. It's the actual process I follow, not a simplified 3-prompt trick.

Most people throw their resume at an LLM and ask it to "make it better." That produces generic fluff. This guide gives you a repeatable loop that builds relevance, tightens signal, and makes sure both machines and humans want to keep reading.

---

## Before You Start

**Grab a clean, ATS-friendly base template.** Fancy columns, icons, and graphics get mangled by applicant tracking systems. Start simple.

The template I use: [SheetsResume](https://sheetsresume.com/resume-template)

**Attach your resume(s) and the full job description** into a new chat. Both need to be in context for every prompt below. If you have multiple versions of your resume tailored to different roles, attach all of them — one of the most valuable early steps is picking the right base and pulling language from the others.

---

## Why This Process Exists

Resumes now pass through two readers: an ATS parser and a human.

AI is on both sides of the table. Candidates use it to write resumes, and hiring teams use it to screen them. This changes the game in two ways. First, keyword coverage and document structure matter more than ever because machines parse your resume before a human sees it. Second, when a hiring manager eventually reads it, it still needs to sound like a person with real experience wrote it — not a prompt.

The tension your resume has to navigate: **optimize for machines first, but write for humans.**

This process handles both.

---

## On Resume Length

The old "one page only" rule made sense when recruiters spent 6 seconds skimming a printed stack. Now ATS systems parse the full document and recruiters read on screens.

The real constraint isn't page count — it's signal density. A tight 2-page resume with zero filler beats a cramped 1-pager where you've gutted your best evidence to fit. Aim for 1–2 pages. Every line should earn its space. If cutting to one page means removing your strongest role-fit evidence, don't cut.

---

## Phase 1: Gap Analysis

Before rewriting anything, you need to know where you stand.

**Prompt:**

> Act as a senior recruiter. Compare my resume against this job description. Give me:
> 1. A match score out of 100
> 2. The top 5 missing keywords, skills, or qualifications
> 3. The weakest bullets or sections
> 4. Any experience that should be reframed for this role
>
> Summarize by ranking which resume would serve as the best base, and outline which components, bullets, words, experiences, framing, and style from each resume are worth extracting.

If you attached multiple resume versions, this step also tells you which one to build from and what to pull from the others. You're not starting from scratch — you're assembling the strongest possible base.

**A tip:** I often run the same prompts in both Claude and ChatGPT, then cherry-pick the best outputs from each. Similar to the idea behind [llm-council](https://github.com/karpathy/llm-council) — I'll copy results from the LLM I like better and ask the other one to review, flag disagreements, and suggest improvements.

---

## Phase 2: Strategic Framing

This is the step most people skip, and it's where the real tailoring happens.

The gap analysis tells you what's missing. Strategic framing decides **how to position yourself** — what identity the resume should project, what narrative arc to follow, and what ordering tells the right story for this specific role.

**Prompt:**

> Based on the gap analysis, what identity should this resume project for this role? Which of my experiences should lead? What's the strongest narrative arc? What framing changes would make the biggest difference — not just keywords, but how the reader perceives my background?

This is the difference between a local optimization (rewriting bullets with better keywords) and a global restructure (changing which experiences lead, how sections are ordered, and what story the first page tells).

For example, the same person might lead with "ML Engineer who ships production systems" for one role and "Applied Scientist who evaluates and builds reliable AI systems" for another. Same experience, completely different framing.

---

## Phase 3: Surface Unlisted Work

Before rewriting, ask yourself: **do I have projects, contributions, or results that aren't on any version of this resume?**

The gap analysis might reveal that your best-fit evidence is something you haven't written up yet. A side project, an internal tool, a cross-functional initiative, a production system you led but never turned into a bullet — these are often the highest-impact additions.

This isn't about fabricating experience. It's about recognizing that your resume is an incomplete sample of your actual work, and the gap analysis can tell you which missing pieces would matter most.

---

## Phase 4: Rewrite for Impact

Now rewrite with a clear strategy, not just better keywords.

**Prompt:**

> Rewrite my experience section following the strategic framing we discussed. Use the Google XYZ formula for every bullet. Keep it truthful, specific, and results-oriented. Do not fabricate metrics, tools, or scope — if I haven't provided a number, flag it and ask me. Use strong action verbs to lead each bullet. Naturally incorporate the missing keywords from the gap analysis.

The Google XYZ formula: "Accomplished [X] as measured by [Y] by doing [Z]." Every bullet should show what you did, how it was measured, and what action drove it.

After the first rewrite, review critically. The most common failure mode is **keyword stuffing** — bullets that sound like a job description instead of a person. Read each bullet out loud. If it sounds like something a human would say in an interview, keep it. If it sounds like an ATS optimization exercise, rewrite it.

---

## Phase 5: Signal-to-Noise Optimization

This is where most AI-assisted resumes plateau. You have strong bullets, good keywords, the right framing — but the resume is too long or too dense because you haven't cut anything.

The hard part: **removing good bullets that are noise for this specific role.**

**Prompt:**

> Review the current resume for signal-to-noise ratio against this specific job description. Which bullets, while strong on their own, are not among the highest-signal evidence for this role? What should be cut, compressed, or merged? What's earning its space and what isn't?

This is where you make hard tradeoffs. A bullet about infrastructure-as-code might be excellent for a platform engineering role but noise for an applied science one. A bullet about stakeholder communication might be essential for a product role but expendable for a deep technical one.

The goal is not the shortest resume. It's the densest relevant resume — where every line maps to something the hiring manager is looking for.

---

## Phase 6: Add Metrics

After the structure is right, go back through and add concrete numbers to any bullet that's still describing a responsibility instead of a result.

**Ask yourself for each bullet:**

- How big was it? (scale, volume, users, documents)
- How good was it? (accuracy, F1, error reduction)
- How fast? (time saved, cycle reduction)
- What changed? (before vs. after)

You don't need metrics on every bullet, but your top 3–4 achievements should answer at least one of these questions with a real number.

---

## Phase 7: ATS Structural Audit

This goes beyond surface-level formatting checks. You want to verify the actual document structure that an ATS parser will see.

**Prompt:**

> Act as an applicant tracking system parser. Scan my resume document and flag any sections, formatting choices, or structural issues that an ATS bot would struggle to parse or classify correctly. Check for: heading styles in the XML, list formatting, hyperlinks, section names, and date parsing.

Common issues that surface-level checks miss:

- **No heading styles applied** — your section headers look like headings to a human but are just bold text to a parser. Use Word's Heading 1 and Heading 2 styles.
- **Unicode bullets instead of Word list formatting** — a `•` character looks right but doesn't create a parseable list structure.
- **Missing hyperlinks** — plain text URLs for email, GitHub, and LinkedIn won't get classified as contact/profile fields.
- **Non-standard section names** — "Selected Research & Projects" is harder for a parser than just "Projects."

---

## The Iteration Loop

Don't stop after one pass through these phases. The process is iterative — a rewrite might reveal a new framing opportunity, cutting a bullet might create space for a stronger one, adding metrics might make a bullet too long and require trimming.

Go back and forth between phases until the resume is both strong for the role and structurally clean.

Then do a final cold validation:

1. **Open a brand-new chat** (Claude or ChatGPT). A fresh chat has no memory of your editing session, so it judges the resume cold.
2. Attach your latest resume and the job description.
3. Use: *"Act as a senior recruiter. Score this resume against the job description out of 100 and explain the score briefly."*
4. Check the score against the table below.
5. Repeat until you land in the sweet spot.

---

## Scoring Guide

| Score | What It Means | Action |
|-------|--------------|--------|
| Below 70 | Major gaps in keywords or relevance | Heavy Phase 1–4 work needed — you may be missing real experience, not just wording. Consider building new projects. |
| 70–79 | Solid foundation, real gaps remain | Targeted Phase 2–5 tweaks |
| 80–84 | Strong — review what's missing, decide if it matters | Minor signal-to-noise and metrics work |
| **85–90** | **Sweet spot — clearly tailored, still sounds human** | **Ship it** |
| Above 90 | Likely over-fitted to the job description | Read it out loud — if it sounds like a job description instead of a person, dial back the keyword density |

These scores come from an LLM role-playing as a recruiter, not a standardized system. Don't treat 84 vs 86 as meaningfully different. The point is a rough compass, not a precise measurement.

---

## Phase Summary

| Phase | What You're Doing | Key Question |
|-------|------------------|--------------|
| 1. Gap Analysis | Finding what's missing and picking the best base | Where do I stand? |
| 2. Strategic Framing | Deciding what identity and narrative to project | Who am I for this role? |
| 3. Surface Unlisted Work | Checking if your best evidence isn't on the resume yet | What have I done that I haven't written up? |
| 4. Rewrite for Impact | Rebuilding bullets with the right framing and keywords | Does every bullet show what I did, how well, and why it matters? |
| 5. Signal-to-Noise | Cutting good-but-irrelevant content | Is every line earning its space for this specific role? |
| 6. Add Metrics | Making results concrete and measurable | How big, how good, how fast, what changed? |
| 7. ATS Audit | Verifying structural parseability | Will the machine read this correctly? |

---

## Final Notes

This process will not create new experience. If you're genuinely lacking experience for a role, you need to go build things — no amount of prompt engineering fixes an empty resume.

The good news: once you've done this for one role, it gets much faster for similar ones. A data scientist posting at Company A and Company B will overlap heavily, and you can skip most of the strategic framing work the second time around. The phases that take the longest the first time — gap analysis, framing, surfacing unlisted work — produce artifacts you can reuse.

*The goal isn't to trick an ATS. It's to make sure the system doesn't throw away a resume that a human would want to read.*