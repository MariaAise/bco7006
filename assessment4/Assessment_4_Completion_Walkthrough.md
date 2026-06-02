# Assessment 4 — Completion Walkthrough

*Business Analytics Solution Modelling: Research Report (BCO7006). A step-by-step guide to completing the report. This walkthrough explains how to interpret each required section and what a strong answer looks like. It follows the official report structure exactly — it does not replace the assessment brief.*

---

## Before you start: four things to get straight

**1. What "your program" means.** You are *not* writing code for this assessment. You are analysing a program someone else already wrote — the Kaggle notebook your group was allocated. Everywhere the brief says "your program," read it as "the notebook your group was given." You explain and evaluate it; you do not build it.

**2. What "the scenario" means.** Your scenario is the Kaggle competition your group was assigned. The competition page describes a real business problem; the notebook is one attempt to solve it. You received two links: the competition (the problem) and the code (one solution).

**3. Who you are writing for.** The report body is for a **non-technical reader** — a manager who wants to know what problem was solved, what was found, and why it matters. They do not want to read code. So the line-by-line code explanation does **not** go in the report body. It goes in a **Technical Appendix** at the end.

**4. The word count and the appendix.** The 2000-word limit covers the report body only. Like references, the Technical Appendix (the copied code plus your explanation of it) sits **outside** the word count. This is what makes the report achievable: plain-language business writing in the body, technical detail quarantined in the appendix.

> **Scope of what you explain.** This unit taught Python fundamentals: variables, lists, dictionaries, strings, loops, if/else, functions, file reading, and the pandas/numpy methods used for handling data. You are expected to explain the parts of the notebook that use **these** constructs — data loading, cleaning, filtering, the loops and conditions, and the pandas/numpy methods that do the analysis. You are **not** expected to explain the internal mathematics of any machine-learning model the notebook may use (e.g. how a decision forest or neural network works). If a cell trains a model, describe *what it does and why* in one or two plain sentences — that is enough.

---

## The report, section by section

The report contains the sections below, in this order. Headings (a)–(i) match the assessment brief.

### (a) Executive summary

A short, standalone overview a busy manager could read on its own. Write this **last**, once you know your insights.

Cover, in plain language:
- The business problem (one or two sentences).
- What the notebook does to address it.
- The one or two most important findings.
- The bottom-line benefit to the business.

Aim for roughly 150 words. No code, no jargon.

### (b) Introduction

A brief overview of the problem behind your scenario.

Answer:
- What is the business problem the competition is trying to solve? State it in business terms, not data-science terms (e.g. "predict which products a customer is likely to buy next," not "minimise MAP@12").
- Who in a real business would care about this, and why?
- What question will the rest of the report answer?

This is where you make clear, once, what your scenario and dataset are.

### (c) Methods

This section explains *how* the notebook works. **Put the detail in the Technical Appendix** (see template at the end) and keep the body brief.

In the **report body**, write a short plain-language paragraph: what data the notebook uses, the main steps it goes through (load → clean → explore → analyse), and the main tools it relies on (pandas for handling the data, numpy for calculations, a plotting library for charts). A non-technical reader should understand the *shape* of the work without seeing a single line of code.

In the **Technical Appendix**, for each code cell you choose to explain:
- Copy the cell.
- Below it, explain: which libraries it uses and why; what any loop is repeating and over what; what any if/else is deciding; and what each pandas/numpy method returns or changes.

You do not have to explain every cell — explain the cells that load, clean, and analyse the data (the parts built from what we covered). Skip or one-line any cell that only trains or tunes a model.

> See the companion notebook `Assessment_4_Methods_Example.ipynb` for one fully worked cell, explained the way we want yours to read.

### (d) Insights

What did the analysis actually find?

- What patterns or relationships did the notebook reveal? Point to specific outputs — a chart, a table, a correlation, a count.
- What is the answer to the question the competition poses?
- Translate each finding into a sentence a manager would understand. "Most customers buy from only two or three product groups" beats "the purchase matrix is sparse."

Refer to the notebook's outputs by what they show, not by cell number.

### (e) Benefits

How would building and using this program help the business?

- What decisions could the business make better or faster with these findings?
- What does the chosen approach do well for this problem (e.g. handles a large dataset, surfaces patterns a human would miss, repeatable as new data arrives)?
- Tie it back to the organisation behind the data (a retailer, a property market, a stock investor, etc.).

### (f) Testing

How did the author check that the program gives correct results under different conditions?

Look in the notebook for things like: checking the size/shape of the data after loading, handling missing values, separating training from testing data, sanity-checking outputs, or guarding against bad input with conditions. Describe what you find.

If the notebook does **little** explicit testing, say so plainly — that is a valid and useful finding — and suggest one or two checks that *could* be added (e.g. confirming no missing values remain, or checking results hold on a different slice of the data). Remember "your program" here means the allocated notebook.

### (g) Division of Work

A short table: each member and what they were responsible for. If your collaboration hit issues (uneven contribution, missed handoffs, etc.), report it here honestly and briefly. This section is about accountability, not blame.

| Member | Main responsibilities / contribution |
|--------|--------------------------------------|
| Name 1 | |
| Name 2 | |
| Name 3 | |
| Name 4 | |

### (h) Conclusion

Recap the problem, the key insight, and the benefit in a few sentences. Note one limitation of the analysis (what it can't tell you, or where the data is thin). Leave the non-technical reader with one clear takeaway.

### (i) References

List the competition link, the notebook link, and any other sources (e.g. library documentation, articles). Use any consistent referencing style and include in-text citations where you draw on a source. The GitHub link and references are **not** part of the presentation.

---

## Pre-submission checklist

- [ ] Report body is under 2000 words (appendix and references excluded).
- [ ] Report body contains no raw code — all code lives in the Technical Appendix.
- [ ] Every section (a)–(i) is present and in order.
- [ ] Executive summary reads on its own, with no jargon.
- [ ] Insights point to specific outputs from the notebook, in plain language.
- [ ] Methods explanation covers data loading, cleaning, loops, conditions, and pandas/numpy methods — and does **not** try to explain model internals.
- [ ] "Your program" / "the scenario" consistently mean the allocated notebook / competition.
- [ ] Division of work table is filled in.
- [ ] Competition link and notebook link are in the references.
- [ ] File named per the brief's convention and uploaded to the correct VU Collaborate dropbox (notebook **and** report).

---

## Technical Appendix — template for explaining a code cell

For each cell you explain, use this pattern:

> **Code**
> *(paste the cell here)*
>
> **What it does**
> *One sentence: the purpose of this cell.*
>
> **Libraries used**
> *Which library each call comes from, and what that library is for.*
>
> **Loops / conditions**
> *If there is a loop: what it repeats and over what. If there is an if/else: what it is deciding.*
>
> **pandas / numpy methods**
> *Each method and what it returns or changes (e.g. `.isnull().sum()` counts missing values per column).*

Repeat for each cell. Explain the data-handling cells thoroughly; for any model-training cell, one or two plain sentences is enough.
