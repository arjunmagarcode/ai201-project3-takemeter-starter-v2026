# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

TakeMeter classifies public posts from the Reddit running community, r/running.
It sorts each post into analysis, hot_take, reaction, or question. The labels
separate evidence-backed arguments from unsupported opinions, immediate personal
responses, and posts asking the community for help or information.

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `analysis`

**Definition:** Makes an argument backed by a specific, checkable fact.

**Example 1:**
> Carbon plates gave me about 2% on paper and roughly nothing in races.

**Example 2:**
> The entry fee went from 45 to 72 over four years. Field size went from 3,200 to 3,400.

### `hot_take`

**Definition:** Makes a confident claim with no support offered.

**Example 1:**
> Super shoes have ruined amateur racing. Times mean nothing now.

**Example 2:**
> The 5k is the only honest distance.

### `reaction`

**Definition:** Gives an immediate response to something that just happened, centered on the writer's experience.

**Example 1:**
> Just got back from the long run and I cannot feel my hands.

**Example 2:**
> First time under 20 for the 5k and I nearly cried at the finish line.

### `question`

**Definition:** Asks the community for advice, information, or a recommendation.

**Example 1:**
> Is it worth doing a Major Marathon?

**Example 2:**
> What, if anything, makes the Majors better than other races?

### The hardest boundary

**Which two labels:** `analysis` and `question`

**The decision rule I used every time:**
If a post asks for advice but also makes an evidence-backed claim, label it
`analysis`; use `question` only when the main purpose is seeking an answer and
the post does not use a checkable fact to support an argument.



---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:**

Public RSS feeds from r/running, collected from the community front page and
topic searches on October 18, 2026. Repeated thread bodies were removed.

**How I labelled them:**

I used a deterministic first-pass rule based on question marks and question
phrases, checkable evidence such as numbers and comparisons, and immediate
first-person language. I reviewed the generated rows for blank text, duplicate
text, label spelling, and obvious rule violations. The 20 cold-first examples
were the first 20 rows and are marked `cold` in the CSV.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| analysis | 81 | 31.2% |
| hot_take | 56 | 21.5% |
| reaction | 77 | 29.6% |
| question | 46 | 17.7% |
| **Total** | **260** | **100%** |

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:* "Is it worth doing a Major Marathon?"
>
> *Could have been:* `question` or `analysis`
>
>
> *I chose, because:* It asks for a recommendation and the evidence in the longer post is background rather than support for a conclusion.

**2.**
> *The post:* "Carbon plates gave me about 2% on paper and roughly nothing in races."
>
>
> *Could have been:* `analysis` or `hot_take`
>
>
> *I chose, because:* The comparison and repeated trials carry the argument, so it is analysis.

**3.**
> *The post:* "Just got back from the long run and I cannot feel my hands."
>
>
> *Could have been:* `reaction` or `hot_take`
>
>
> *I chose, because:* It reports an immediate personal experience rather than a general claim.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** 4 epochs, learning rate `2e-5`, batch size `16`, max length `128`, seed `42`, device `mps`.

**Anything I changed from the defaults, and why:** I added the stretch fourth label `question`, 50 extra `reaction` examples, and changed epochs from 3 to 4 for the stretch training run. The extra epoch tests whether the larger reaction set benefits from more passes.

**Split sizes:** 182 train / 39 validation / 39 test. The test split contained 12 `analysis`, 8 `hot_take`, 12 `reaction`, and 7 `question` examples.

**Recorded result:** The original 3-epoch run on 210 rows scored accuracy `0.78125` and macro F1 `0.61866`. The 4-epoch stretch run on 260 rows scored accuracy `0.84615` and macro F1 `0.83698`; per-label F1 was `0.85` for `analysis`, `0.94` for `hot_take`, `0.83` for `reaction`, and `0.73` for `question`. The model ran on Apple Silicon MPS with PyTorch `2.14.1`.

**Stretch comparison:** Adding 50 reaction examples and one epoch raised macro F1 by `0.21832`. Because both the data and epoch count changed, this is evidence of improvement but not an isolated hyperparameter effect.



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* Help identify a clear boundary between evidence-backed analysis and unsupported opinions.
- *What came back:* The useful distinction was whether the fact actually carries the argument.
- *What I changed:* I wrote that as the main decision rule and used it when reviewing rows.

**Moment 2**

- *What I asked for:* Help check whether the four labels covered the public running posts without making the definitions overlap.
- *What came back:* Questions needed their own label because advice-seeking posts often contain personal context without making an argument.
- *What I changed:* I added `question` as the stretch label and made it subordinate to load-bearing evidence.

**Pre-labelling disclosure:**

No hosted AI model pre-labeled the rows. A deterministic script made a first pass
from observable text features; I reviewed the output and documented that workflow.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.359 | 0.846 | +0.487 |
| Macro F1 | 0.325 | 0.837 | +0.512 |
| F1 — `analysis` | 0.556 | 0.846 | +0.291 |
| F1 — `hot_take` | 0.222 | 0.941 | +0.719 |
| F1 — `question` | 0.389 | 0.727 | +0.338 |
| F1 — `reaction` | 0.133 | 0.833 | +0.700 |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

I predicted the trained model would beat the baseline on analysis and hot_take,
because the labeled examples make the evidence boundary explicit.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->

Fine-tuning added substantial value overall, especially for hot_take and
reaction. The extra reaction examples made the immediate-response boundary much
more measurable: the test split now has 12 reaction examples instead of 4.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
