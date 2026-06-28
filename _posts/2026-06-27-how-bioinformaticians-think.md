---
layout: post
title: "How Bioinformaticians Think: Asking the Right Questions Before Running Any Tool"
date: 2026-06-27
description: "Anyone can run a pipeline. Type the command, wait for the job to finish, get a table of results. The hard part of bioinformatics has never been execution — it is deciding what question the pipeline should actually answer. This post is not a tutorial. It is about the thinking that happens before the first command is typed: how to turn a vague curiosity into a testable question, and two underrated sources of analytical maturity that nobody talks about — reviewing manuscripts and reading GitHub issues."
comments: true
giscus_comments: true
featured: true
permalink: /blog/how-bioinformaticians-think/
tags:
  [
    bioinformatics,
    scientific-thinking,
    study-design,
    peer-review,
    GitHub-issues,
    career-development,
    research-methodology,
    critical-thinking,
    metagenomics,
    beginners,
  ]
---

🧬 _Day 96 of Daily Bioinformatics from Jojy's Desk_

Anyone can run a pipeline. Install the tool, point it at your FASTQ files, wait for the SLURM job to finish, get a table of results. This is true whether the pipeline is Kraken2 or DESeq2 or AlphaFold. The execution step has gotten easier every year — better documentation, conda environments, one-line installs, Docker containers that handle every dependency for you.

What has not gotten easier, and never will, is deciding what question the pipeline should actually answer. This is the part of bioinformatics that takes years to develop, that no software update will automate, and that separates an analysis that quietly produces a table from an analysis that actually advances understanding.

This post is not a tutorial. There is no code here. It is about the thinking that happens before you type the first command.

---

## The question behind the question

Most bioinformatics training focuses on the "how." How do you run Kraken2. How do you interpret a DESeq2 results table. How do you build a co-occurrence network. These are necessary skills, but they are answers to a question that was already decided for you — usually by an advisor, a paper you are replicating, or a default workflow you found online.

The harder and more valuable skill is recognizing when the question itself is too shallow to be useful.

**Instead of:** _Which taxa are present in my samples?_

**Ask:** _Which taxa are driving the difference I observe between treatments?_

The first question is answered by any taxonomic classifier. It produces a list. The list is not wrong, but it is not interpretable on its own — every sample has taxa present, that fact alone tells you nothing about your experiment. The second question forces you to define what "difference" means (statistical? ecological? both?), forces you to choose an appropriate differential abundance method, and forces you to think about whether the taxa you find are biologically plausible drivers or just statistical artifacts of an unbalanced design.

**Instead of:** _Which genes are present in my metagenome?_

**Ask:** _Which genes explain the ecological pattern I already observed?_

Again — the first question is satisfied by any annotation tool. Run Prokka, run eggNOG-mapper, get a functional profile. But "present" is almost meaningless in isolation. Almost every gene family is present in almost every reasonably complex metagenome at some level. The question that actually moves your research forward is connective: you saw something — a community shift across a salinity gradient, a bloom in a particular phylum, a change between FL and PA fractions — and you are asking which genes, mechanistically, could explain it.

This reframing changes everything downstream. It changes which statistical test you choose, which comparisons you make, which figures you build, and most importantly, what counts as a meaningful result versus noise.

### A simple test for your own question

Before running any tool, try finishing this sentence: _"If this analysis works perfectly, the result will tell me \_\_\_."_

If you cannot finish that sentence with something specific and falsifiable, the analysis is not ready to run yet — no matter how excited you are to see the output. "It will tell me what's there" is not a finished sentence. "It will tell me whether genome size correlates with particle-association after controlling for phylum" is.

---

## Lesson 1: Reviewing manuscripts taught me to ask better questions

The single biggest improvement in my analytical thinking did not come from a course or a textbook. It came from reviewing manuscripts for journals.

When you review a paper, you are forced into a specific mental posture that is completely different from the posture you have when running your own analysis. As a reviewer, you naturally ask:

- Is the analysis appropriate for this type of data?
- Is there another explanation for this result that the authors haven't considered?
- What controls are missing?
- Is this result statistically significant but biologically meaningless?
- Does the conclusion actually follow from the data, or is there a gap?

The strange thing is that these questions are not hard to ask. Any trained reviewer asks them automatically, almost reflexively, about someone else's work. What is hard is asking them about your _own_ work, in real time, before you have already committed to a narrative.

Reviewing teaches you to internalize that skeptical voice and turn it inward. After you have reviewed a dozen papers and seen a dozen ways that a reasonable-looking analysis can have a hidden flaw — an unaccounted confounder, a multiple-testing problem, a sample size too small to support the claimed effect — you start running that same checklist on your own pipeline before you trust the output.

A practical exercise: the next time you get a result you are excited about, write a one-paragraph peer review of your own analysis as if a stranger had submitted it to you. Be as skeptical as you would be of someone else's work. You will usually find at least one question worth answering before you move forward.

---

## Lesson 2: GitHub issues are an underrated education

This is something almost nobody talks about, and it deserves more attention than it gets.

When you are learning a tool from documentation alone, you see the idealized version: clean example data, expected output, a tutorial that works on the first try. What you do not see is the tool's actual behavior in the messy, real-world conditions your own data will present.

This is exactly what GitHub Issues capture. The typical pattern looks like this:

```
Tool works perfectly on the documented example
        ↓
A real user runs it on their own (messier) data
        ↓
User reports a weird, unexpected, or seemingly wrong result
        ↓
The developer responds, explaining a limitation, an edge case,
or an assumption baked into the tool that isn't obvious from
the documentation
        ↓
You read this exchange and learn something the manual never told you
```

This is gold, and it is free, and it is sitting in the open Issues tab of every actively maintained bioinformatics tool you use.

A few concrete examples of the kind of thing you learn only from issues:

- A tool's default parameter assumes paired-end reads of a certain length range, and silently produces degraded results outside that range — never stated explicitly in the docs, but explained clearly in a three-comment thread from 2022
- A statistical method's p-values become unreliable below a certain sample size, something the developer mentions in response to a confused user but does not put in a warning message
- Two parameters interact in a non-obvious way, and changing one without the other produces a result that looks plausible but is wrong — caught only because someone reported it and the maintainer explained the interaction

None of this is information you would find by reading the methods paper or the README. It is the accumulated, hard-won troubleshooting knowledge of every person who has hit an edge case before you — and the tool author's honest explanation of why.

### How to actually use this as a learning resource

Before you commit to using a tool for an important analysis, spend fifteen minutes in its GitHub Issues tab (and the Closed issues, not just Open ones — that is where resolved confusions live):

- Search for your specific use case (e.g., "low coverage," "small sample size," "metagenome mode")
- Read a handful of issues where the user got a confusing result and the developer explained why
- Note any caveats that change how you should interpret your own output

This single habit will save you from more silent analytical mistakes than almost anything else you can do in fifteen minutes.

---

## What this looks like in practice

Put together, here is roughly what a more mature analytical workflow looks like, compared to a more naive one:

| Naive approach                                       | More mature approach                                                             |
| ---------------------------------------------------- | -------------------------------------------------------------------------------- |
| Run the tool everyone uses                           | Ask what question you actually need answered, then pick the tool that answers it |
| Accept the default parameters                        | Check the tool's GitHub issues for known caveats at your data scale              |
| Report whatever is statistically significant         | Ask whether the result is also biologically and ecologically plausible           |
| Trust the first result that confirms your hypothesis | Review your own analysis the way you'd review someone else's submission          |
| Treat "what is present" as the end goal              | Treat "what explains the pattern I observed" as the actual goal                  |

None of this replaces technical skill. You still need to know how to run the tools, interpret the statistics, and write the code. But technical skill without this layer of questioning produces a researcher who can execute pipelines competently and still draw the wrong conclusions from them — confidently.

---

## The habit worth building

If there is one habit to take from this post, it is this: before you open a terminal, write down — in plain language, no jargon — what question you are trying to answer and what result would make you change your mind. Then go run the pipeline.

It takes thirty seconds. It will save you from more wasted analyses than any new tool you could learn this year.

---

_Posted as Day 96 of Daily Bioinformatics from Jojy's Desk._

![See your plot](/assets/img/bionfo.png)
