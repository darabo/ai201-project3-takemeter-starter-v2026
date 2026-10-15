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

TakeMeter sorts public, non-NSFW question submissions from r/NoStupidQuestions.
Each example combines the submission title and body, but excludes comments,
deleted posts, and megathreads. The classifier assigns every question to one
of four intent-based labels: `explanation`, `practical_guidance`,
`personal_advice`, or `social_judgment`.

I read recent posts before choosing these labels so that the distinctions came
from the community's actual questions rather than from a topic list I invented
in advance. The goal is to distinguish what kind of answer a question is
seeking, not whether the question is good, important, or well written.

### Stretch work declared before building

- I will use a fourth label, `social_judgment`, because questions about what
  people or society consider normal or acceptable are distinct from the other
  three kinds of requests.
- After the initial labelled set is complete, I will add 50 examples to the
  thinnest label and document how its count changed.
- I will run a second training run with exactly one changed hyperparameter and
  document what changed and which reported numbers moved.



---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `explanation`

**Definition:** A question that asks for a factual explanation, identification,
or clarification about a fact, system, institution, or observable phenomenon,
rather than asking what the author should do or how people should judge it.

**Example 1:**
> Why do babies cry immediately after they're born? Like, what actually makes
> them start crying?

**Example 2:**
> Why do successful top-level football managers not take jobs at lower-league
> clubs later in their careers?

### `practical_guidance`

**Definition:** A question that asks for a method, recommendation, or action
to solve a concrete practical or technical problem.

**Example 1:**
> Best Ways/Tools to Safely Keep Track of Passwords

**Example 2:**
> TikTok question regarding messages and chats: if you block a person, does it
> delete all previous conversations with that person?

### `personal_advice`

**Definition:** A question about the author's own emotional, social, or
interpersonal situation that asks what they should do, say, or change.

**Example 1:**
> Is always being the bigger person actually helpful in relationships?

**Example 2:**
> I look scary and miserable apparently, any advice on how to fix this?

### `social_judgment`

**Definition:** A question that asks whether a behavior, belief, or social
norm is common, acceptable, justified, or preferable in a broader social
sense.

**Example 1:**
> Can someone live an entirely fulfilling life without ever experiencing
> romantic love?

**Example 2:**
> Should all social media apps have a setting to not show any AI content at
> all, if you don't want to see it?

### The hardest boundary

**Which two labels:** `personal_advice` and `social_judgment`

**The decision rule I used every time:** If the central request is what the
author should do in their own specific situation, use `personal_advice`. If it
primarily asks how people or society view a behavior or belief, use
`social_judgment`, even when the author includes a personal example.



---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Public submissions from
r/NoStupidQuestions. Each row combines a post title and body; comments and
author information were not collected. I excluded NSFW, deleted, and
megathread posts. I first tried Reddit's public feed; when that required a
human-verification challenge, I retrieved the same public submissions from
the Arctic Shift public Reddit archive instead.

**How I labelled them:** I labelled the first 20 examples cold, without
AI help, and marked them `cold review` in `labels.csv`. The other 286 rows
were prelabelled by an AI tool that I gave my label definitions. I then had
every one of those rows checked in two ways. A second AI pass suggested
corrections to the first 228 (I approved 27 clear ones, such as how-to
questions that had been labelled `personal_advice`, and I deleted two
near-duplicate posts). After I added more posts, two independent AI reviewers
relabelled all 306 rows blind, without seeing the existing labels. They
disagreed with the current label on 51 rows. For the 48 that were not cold
rows, I read each post in full and decided: I changed 32 labels and kept the
other 16, and each row's `note` says which. Rows where I kept the label are
the real boundary cases. I did not change any cold row; three of them (rows 6,
14 and 18) disagreed with the blind reviewers and are kept as my own cold
labels. I have not personally re-read every one of the 286 AI-labelled rows;
their notes say `student has not re-read`.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `explanation` | 76 | 24.8% |
| `practical_guidance` | 61 | 19.9% |
| `personal_advice` | 80 | 26.1% |
| `social_judgment` | 89 | 29.1% |
| **Total** | **306** | **100%** |

**Stretch: 50 extra examples for the thinnest label.** Before the extra
examples, the 250 base rows had `social_judgment` 89, `explanation` 76,
`practical_guidance` 61 and `personal_advice` 24, so `personal_advice` was
the thinnest label. I added 56 `personal_advice` rows (marked `stretch +50` in
`labels.csv`), which took it from 24 to 80 and from 9.6% to 26.1% of the file.
I had originally added 50 `social_judgment` rows as this stretch item because
that label looked thinnest on the AI prelabels (37 of the first 200 rows).
After the label corrections changed which label was thinnest, I replaced that
plan: the earlier 50 rows are kept as ordinary data, and their notes say so. I
collected 58 posts for the new stretch, and two were relabelled in review, so
56 count. What happens to the `personal_advice` scores will be in the next
unit's results.

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:* "Is it normal to completely 'forget' how to walk naturally when
> you suddenly realize someone is walking behind you? Whenever I'm walking down
> the street and I notice someone close behind me my brain suddenly panics and
> forces me to manually control my legs. I feel like an alien trying to simulate
> human walking and I'm convinced I look incredibly stupid. Does this happen to
> anyone else or am I broken?"
>
> *Could have been:* `personal_advice` or `social_judgment`
>
> *I chose:* `social_judgment`, because the author never asks what to do. The
> post describes a personal experience, but the question it ends on is whether
> it is common ("does this happen to anyone else"). My decision rule for this
> boundary says that when the central request is how people experience
> something, it is `social_judgment` even if the author gives a personal
> example.

**2.**
> *The post:* "Is it normal to not know how to cook basic meals in your 20s?
> I've always relied on takeout or my parents, but I'm trying to learn now.
> Where should a complete beginner start?"
>
> *Could have been:* `social_judgment` or `practical_guidance`
>
> *I chose:* `practical_guidance`, because the post ends with a concrete
> request for a method ("where should a complete beginner start?"). The "is it
> normal" opening is a reassurance question, and an AI prelabel had called the
> whole post `social_judgment` from it. I changed it in review. The rule I now
> use is that when a post asks both whether something is normal and how to do a
> practical task, the practical request decides the label.

**3.**
> *The post:* "Why is it acceptable for cops to run stop signs WITHOUT lights or
> sirens but if I roll aggressively slowly to 1 mph I get pulled over? How many
> times have you seen a cop do something illegal but you can't do anything about
> it? [...] I once followed a cop on the highway doing 110mph without his lights
> or sirens on and when they got off the exit I just looked at them like wtf."
>
> *Could have been:* `explanation` or `social_judgment`
>
> *I chose:* `social_judgment`, because "why is it acceptable" asks whether
> a double standard is justified, not what causes it. An `explanation` version
> of this post would ask why police are exempt from a traffic rule, or how the
> exemption works. One of my two blind AI reviewers read it as `explanation`
> because it starts with "why". I keep every "why is it acceptable" question
> as `social_judgment` and every "why does X happen" question as `explanation`.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:**

**Settings:** <!-- epochs, learning rate, batch size, seed -->

**Anything I changed from the defaults, and why:**

**Split sizes:** <!-- train / val / test, and per-label counts in the test
split. If a label had fewer than about 8 in test, say so — it explains a lot
of next unit's variance. -->



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Moment 2**

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Pre-labelling disclosure:**

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



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
