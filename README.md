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

**What this label captures that the other three miss:** questions that ask
whether something is normal, common, acceptable or justified ("is it normal
to...", "why is it acceptable..."). They ask for no factual explanation, no
method, and no decision about the author's own life, so `explanation`,
`practical_guidance` and `personal_advice` all fit them badly, and a large share
of r/NoStupidQuestions posts are of this kind.

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
were prelabelled by an AI tool that I gave my label definitions. Those labels
then went through three checks. First, a second AI pass suggested corrections
to the first 228 rows; I approved 27 clear ones (such as how-to questions that
had been labelled `personal_advice`) and deleted two near-duplicate posts.
Second, after I added more posts, two independent AI reviewers relabelled all
306 rows blind, without seeing the existing labels. They disagreed with the
current label on 51 rows. For the 48 that were not cold rows, I read each post
in full and decided: I changed 32 labels and kept the other 16, and each row's
`note` says which. The rows where I kept the label are the real boundary
cases. I did not change any cold row; three of them (rows 6, 14 and 18)
disagreed with the blind reviewers and are kept as my own cold labels. Third, I
read through every label in `labels.csv` myself and made no further changes,
so no row is still marked as pending review.

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

**Base model:** `distilbert-base-uncased`, fine-tuned for four-way
classification with the notebook in this repo.

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, 128 tokens per post,
seed 42. I trained on an Apple Silicon GPU (device `mps`, "Apple Silicon GPU
(MPS)") with torch 2.14.1; `results.json` records the same device.

**Anything I changed from the defaults, and why:** Nothing. The main run
used the notebook's default settings.

**Split sizes:** The notebook splits `labels.csv` 70/15/15, stratified by
label: 214 train, 46 validation, 46 test (306 rows in total). The test split has
12 `explanation`, 9 `practical_guidance`, 12 `personal_advice` and 13
`social_judgment` posts. No label has fewer than 8 test examples, though
`practical_guidance` is close at 9, so its score will still move between seeds.
`results.json` and `test_split.csv` are committed.

**Second training run (stretch): one hyperparameter changed.** I ran the
same 306 rows again with only the learning rate changed, from 2e-5 to 5e-5.
Epochs (3), batch size (16), 128 tokens, seed 42 and the 70/15/15 split were
unchanged, so both runs were scored on the same 46 test posts. The result is
saved as `results_lr5e-5.json`; the main run stays in `results.json`.

| Measure | Run 1 (lr 2e-5) | Run 2 (lr 5e-5) | Change |
|---|---|---|---|
| Accuracy | 0.652 (30 of 46) | 0.696 (32 of 46) | +0.043 |
| Macro F1 | 0.670 | 0.712 | +0.042 |
| F1 `explanation` | 0.632 | 0.640 | +0.008 |
| F1 `practical_guidance` | 0.750 | 0.889 | +0.139 |
| F1 `personal_advice` | 0.727 | 0.727 | 0.000 |
| F1 `social_judgment` | 0.571 | 0.593 | +0.022 |
| Accuracy, most confident third | 0.800 | 1.000 | +0.200 |
| Accuracy, least confident third | 0.533 | 0.467 | -0.067 |
| Mean confidence when right | 0.341 | 0.550 | +0.209 |
| Mean confidence when wrong | 0.322 | 0.436 | +0.114 |

What moved: `practical_guidance` gained the most (6 of 9 test posts right to
8 of 9), and `explanation` recall rose from 0.50 to 0.67 while its precision
fell from 0.86 to 0.62, because four `social_judgment` posts were now predicted
as `explanation`. `personal_advice` did not move at all (8 of 12 right in both
runs, with the same four posts predicted as `social_judgment`). The biggest
change was confidence. At 2e-5 the model was barely more confident when right
than when wrong (0.34 against 0.32, close to the 0.25 a four-way guess would
give), and at 5e-5 it became more confident overall and its most confident
third was right every time. I think the lower rate left the model undertrained
after 3 epochs on 214 examples, and I have not tested that. These are single
runs on one seed, and one test post is worth 2.2 points of accuracy, so the
overall gain is two more correct predictions and could be noise. The next unit
repeats the comparison across three seeds.

**Extra examples (stretch): 50 more for the thinnest label.** The 250 base
rows had `social_judgment` 89, `explanation` 76, `practical_guidance` 61 and
`personal_advice` 24, so `personal_advice` was the thinnest label. I added 56
more `personal_advice` posts, taking it to 80 of 306 rows. They came from the
same public archive of r/NoStupidQuestions as the rest of the data
(Arctic Shift): I pulled the 1,500 most recent submissions, dropped NSFW,
removed and near-duplicate posts, and an AI tool selected the ones that asked
what the author should do about their own emotional or social situation. I then
reviewed them like every other row. Fifty-eight were collected, and two were
relabelled in review, so 56 count. They are marked `stretch +50` in
`labels.csv`, and `data/labels_base250.csv` is the file without them.

To see what the extra examples did, I trained twice with the default settings
(3 epochs, learning rate 2e-5, batch 16, seed 42): once on the 250 base rows
(`results_250rows.json`) and once on all 306 rows (`results.json`).

| Measure | 250 rows | 306 rows | Change |
|---|---|---|---|
| `personal_advice` rows in the file | 24 | 80 | +56 |
| `personal_advice` posts in the test split | 4 | 12 | +8 |
| F1 `personal_advice` | 0.000 | 0.727 | +0.727 |
| Recall `personal_advice` | 0.000 | 0.667 | +0.667 |
| Precision `personal_advice` | 0.000 | 0.800 | +0.800 |
| F1 `explanation` | 0.588 | 0.632 | +0.044 |
| F1 `practical_guidance` | 0.615 | 0.750 | +0.135 |
| F1 `social_judgment` | 0.619 | 0.571 | -0.048 |
| Accuracy | 0.579 (22 of 38) | 0.652 (30 of 46) | +0.073 |
| Macro F1 | 0.456 | 0.670 | +0.214 |

With 24 examples the model never predicted `personal_advice` at all. All four
`personal_advice` test posts were predicted as `social_judgment`, which also
pulled in 7 `explanation` and 5 `practical_guidance` posts and gave
`social_judgment` a recall of 1.00 but a precision of 0.45. With 80 examples,
the model got 8 of 12 `personal_advice` posts right. `social_judgment` fell a
little (F1 0.619 to 0.571) because the model stopped using it as the label to
guess when unsure. The two runs used different test posts (38 against 46, since
the datasets differ), the `personal_advice` test set in the 250-row run is only
4 posts, and this is one seed, so the size of the change is rough, but a label
going from never predicted to mostly right is not a small effect.

---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1: checking the AI prelabels**

- *What I asked for:* After the AI prelabelled 228 posts using my definitions, I
  asked a second AI pass to read all of them again and suggest which labels
  were wrong.
- *What came back:* It flagged 42 rows, 27 of them clear errors and 15 judgment
  calls. It also found a pattern: the word "setup" had pushed "why" questions
  into `practical_guidance` ("Why are non-profit coding bootcamps setup?"), and
  plain how-to questions (cooking chicken, getting Spotify to recommend more
  varied music) had been labelled `personal_advice`. It found two near-duplicate
  posts the notebook's check would have missed.
- *What I changed:* I approved the 27 clear corrections and deleted one post
  from each duplicate pair. I left the 15 judgment calls for my own review. The
  corrections also changed which label was thinnest (it became
  `personal_advice`), so I replaced my first stretch plan and added `personal_advice`
  posts instead.

**Moment 2: blind relabelling**

- *What I asked for:* I had two independent AI reviewers relabel all 306 posts
  using my definitions and decision rules, without being shown any current label.
- *What came back:* They agreed with the current label on 255 rows (83%) and
  disagreed on 51, including three of my 20 cold rows.
- *What I changed:* For each of the 48 disagreements on non-cold rows I read the
  full post and decided myself. I changed 32 labels and kept 16. For example, I
  moved a GED, cash-job and bank-card question to `practical_guidance` and kept
  "Why is it acceptable for cops to run stop signs..." as `social_judgment`
  even though one reviewer called it `explanation`. I did not change any of
  my cold labels.

**Pre-labelling disclosure:** 286 of the 306 rows in `labels.csv` were first
labelled by an AI tool (Claude), which I gave my label definitions. Only the
first 20 rows were labelled by me without help, and they are marked `cold
review`. The extra 58 `personal_advice` posts for the stretch were found and
selected by the same AI tool from a public archive of r/NoStupidQuestions, and
2 of them were relabelled in review. I reviewed every label in the file myself
after the AI checks described above. An AI coding assistant also drafted the
wording of the Dataset section and the three hard cases from my decisions, and
made two small edits to `criteria.md` (the reason under criterion 3, and a
definition of "confident" in criterion 5); the numbers in my criteria are
unchanged.

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
