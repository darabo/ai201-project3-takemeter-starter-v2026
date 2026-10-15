# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1.

Across all three Unit 6 seeds, overall held-out accuracy is at least **0.60**.

**Why this target:**

With four labels, a uniform guess would be right about 25% of the time. A 60%
target asks the model to learn a useful signal from the wording of questions,
not simply identify that every example came from the same subreddit.



---

## 2.

Across all three Unit 6 seeds, held-out macro F1 is at least **0.55**.

**Why this target:**

`explanation` questions may be more common than the other three intents. Macro
F1 weights `practical_guidance`, `personal_advice`, and `social_judgment`
equally, so this target prevents a strong score on the largest label from
standing in for useful performance on the rest of the taxonomy.



---

## 3.

Across all three Unit 6 seeds, every individual label has held-out F1 of at
least **0.40**.

**Why this target:**

The boundary between `personal_advice` and `social_judgment` is deliberately
subtle, and the smallest label (about 50 of 306 rows) will have only about seven
examples in a 15% held-out split. A 0.40 floor is demanding enough to catch a collapsed
label, while allowing for the volatility of a small test sample.



---

## 4.

In the final `labels.csv`, each label represents at least **15%** and no more
than **40%** of the 306 labelled posts.

**Why this target:**

The four labels describe distinct question intents that appear regularly in
r/NoStupidQuestions, so none should be a token category. Keeping the planned
dataset in this range gives each intent enough examples to learn while avoiding
an `explanation` majority that would encourage the model to guess one label.



---

## 5.

Across all three Unit 6 seeds, ranking held-out predictions by the model's top
softmax probability, the most-confident third has accuracy of at least **0.65** and is at least **0.10** more accurate than
the least-confident third.

**Why this target:**

This classifier is most useful when it can distinguish questions it recognizes
clearly from borderline cases, especially near the `personal_advice` versus
`social_judgment` boundary. A ten-point gap makes confidence an interpretable
signal rather than a number that moves with no practical meaning.



---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
