# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:** The `campus_life` corpus stores each fact in a single short document — one post per topic. If the right document is anywhere in the top-5 results, the answer is there. I allow one miss because my question about the shuttle stop being skipped when the driver is behind is a detail that only appears in one sentence of one document, and a slightly imprecise query could miss it.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:** The grounding instruction in `generate.py` explicitly tells the model to name the document it used, and the prompt assembles each chunk with its filename attached (`[from admin_pass_fail_option.txt]`). There is no reason for this to fail unless the model ignores explicit instructions — which is the very thing the instruction is there to prevent. Five of five is achievable; four of five would mean accepting that the model sometimes ignores a direct rule, and I don't want to accept that.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

**Why this target:** The five `OUT_OF_SCOPE` questions (capital of Mongolia, oil changes, the 1994 World Cup, ibuprofen dosage, Rust loops) are completely unrelated to student life. I expect the gate to catch all five, but allow one miss in case one question happens to share vocabulary with a document (e.g. "loop" appearing in a campus transit description). I'll tighten this to 5 of 5 if the distances show a clean gap in Milestone 4.

---

## 4. Each chunk reads as a complete thought

At least 4 of 5 sampled chunks contain no sentence cut in half at either end — the chunk begins and ends at a natural boundary, and a reader could answer a question from it without needing to read the chunk before or after it.

**Why this target:** The `campus_life` documents are 178–549 characters each, and the default chunk size of 800 characters means almost every document becomes exactly one chunk with no mid-sentence cuts. Reading three documents confirmed this: each post is a self-contained thought. I allow one miss for the rare longer document (like `admin_housing_lottery.txt` at ~480 characters) where an overlap might produce a partial repeat. This criterion is what tells me whether my chunking fits the document shape.

---

## 5. The named source file is the correct one

In at least 4 of 5 answers, the source file the model cites actually contains the answer it gave — not just any file that happened to be retrieved.

**Why this target:** Criterion 2 only checks that *a* source is named. This checks that it is the *right* source. Because `campus_life` stores one fact per file with clear filenames (e.g. `admin_printing_quota.txt`), a grader can open the cited file and confirm the answer is in it. I allow one miss because when multiple files are retrieved (e.g. two housing posts both mentioning laundry), the model may cite a supporting document that contains a related but not primary fact. Four of five is strict enough to catch systematic wrong attribution without penalising a reasonable secondary citation.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->
