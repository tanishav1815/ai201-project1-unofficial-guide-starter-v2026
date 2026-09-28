# The Unofficial Guide

**name: Tanisha Verma ,corpus: 'campus_life' **

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

<!-- Three or four sentences. Which corpus you picked, and the kinds of
     questions your system answers. Write it for someone who has never seen
     this repo.

     Milestone 5. -->

     This system is a retrieval-augmented guide built on the campus_life corpus — 88 short student-written posts covering academic policies, housing, dining, transit, and course advice. You ask a question in plain English and the system finds the most relevant post, then generates a grounded answer that names exactly which file it came from. It answers things like "How late can I declare pass/fail?", "Do dining dollars roll over?", or "How often does the shuttle run?" — practical facts a new student would need but struggle to find on the registrar's site. Questions outside the corpus (anything not covered by student posts) are caught by a relevance gate and refused rather than answered with a guess.

## Chunking Strategy

**Chunk size:** paragraph boundaries (blank lines)
**Overlap:** none

When I read the `campus_life` documents in Milestone 1, I noticed they are short posts averaging 317 characters — far below the 800-character default chunk size. The fallback chunker never split any of them, so 88 documents became exactly 88 chunks, one per document. That's a problem because many posts cover multiple distinct topics in separate paragraphs: one section on the good, one on the bad, one on laundry costs. A question about laundry costs would retrieve the entire document as noise around the one relevant sentence.

The documents are structured with a short title line followed by 1–4 content paragraphs, each separated by a blank line. Splitting on blank lines gives one chunk per thought. I drop any paragraph shorter than 50 characters — the longest title line across all 88 documents is 47 characters, so 50 is a safe threshold that removes headings without touching content.

No overlap is needed: campus_life paragraphs are self-contained. Sentences never run across a blank line, so there is no mid-sentence cut for overlap to repair.

Result: 88 documents → 179 chunks, average 139 characters, shortest 51, longest 373.

## Sample Chunks

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```
You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** — source: `course_cs_340_exams.txt#1` — produced by: `chunker.py::split_documents`

```
Start the term project in week three, not week eight; everyone learns this the hard way.
```

**Chunk 3** — source: `course_phys_130_workload.txt#1` — produced by: `chunker.py::split_documents`

```
It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```

**Chunk 4** — source: `health_center.txt#0` — produced by: `chunker.py::split_documents`

```
Walk-in hours are 8am to 11am; everything after that is by appointment and appointments run about a week out. If something is urgent, go at 8am and wait rather than booking.
```

**Chunk 5** — source: `housing_morrow_house.txt#1` — produced by: `chunker.py::split_documents`

```
The good: cheapest housing tier by about $900 a year, and the singles are real singles.
```

## Sample Answer

**Question:** How late can I declare a course pass/fail?

**Answer:**

```
You can declare a course pass/fail as late as week eight, after you've seen your midterm.

Source: admin_pass_fail_option.txt
```

**My relevance cutoff:** 0.6

The two groups are completely separated with a large gap. In-corpus questions scored 0.22–0.46; out-of-corpus scored 0.82–0.91. The default cutoff of 0.6 sits squarely in that gap — there is no ambiguous zone to worry about. Raising it toward 0.75 would change nothing; lowering it below 0.5 would start refusing the shuttle question (best distance 0.46).

| Question | In corpus? | Best distance |
|---|---|---|
| How late can I declare a course pass/fail? | Yes | 0.2238 |
| Do dining dollars roll over at the end of the year? | Yes | 0.2927 |
| How much free printing does each student get per semester? | Yes | 0.2540 |
| How often does the campus shuttle run on weekdays? | Yes | 0.4624 |
| When does the library close during the regular term? | Yes | 0.3879 |
| What is the capital of Mongolia? | No | 0.8641 |
| How do I change the oil in a diesel engine? | No | 0.9106 |
| Who won the 1994 World Cup? | No | 0.8736 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8243 |
| How do I write a for loop in Rust? | No | 0.8313 |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

     

**1.** I described the structure of the campus_life documents to Claude — short posts averaging 317 characters, each with a title line followed by 1–4 paragraphs. I asked it to write a chunker that splits on blank lines and skips short paragraphs. The first draft used 40 characters as the minimum — I checked the actual documents myself and found the longest title was 47 characters, so I changed the threshold to 50.



**2.**  After setting up the distance table in the README, I asked Claude to help me phrase the cutoff reasoning. It gave me a generic explanation about "two groups with a gap." I rewrote it to include the actual numbers (0.22–0.46 vs 0.82–0.91) and the specific reason I kept 0.6 — lowering it below 0.5 would have refused the shuttle question.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Each chunk reads as a complete thought | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Named source file is the correct one | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

Produced by: `run_eval.py::main`, retrieval by `store.py::search`, chunks from `chunker.py::split_documents`. Full output in `results/run_2026-09-27_2219_before.md`.

**Criterion 1 — retrieved chunk contains the answer (run 1):**

Question: How late can I declare a course pass/fail?
Retrieved: admin_pass_fail_option.txt (distance 0.2238) — chunk contains "as late as week eight" ✓

Question: Do dining dollars roll over at the end of the year?
Retrieved: admin_dining_dollars.txt (distance 0.2927) — chunk contains "whatever is left in May disappears" ✓

Question: How much free printing does each student get per semester?
Retrieved: admin_printing_quota.txt (distance 0.2540) — chunk contains "$30 of printing per semester" ✓

Question: How often does the campus shuttle run on weekdays?
Retrieved: transit_shuttle.txt (distance 0.4624) — chunk contains "every 20 minutes" ✓

Question: When does the library close during the regular term?
Retrieved: housing_morrow_house_noise.txt (distance 0.3879) — chunk contains "open until 2am during term" ✓

**Criterion 2 — every answer names a source (run 1):**

```
You can declare a course pass/fail as late as week eight, after you've seen your midterm.
Source: admin_pass_fail_option.txt

Dining dollars roll over from the autumn semester to the spring semester, but whatever is left in May disappears and does not roll over to the following autumn (admin_dining_dollars.txt).

Each student gets $30 of printing per semester, which is roughly 600 black-and-white pages (admin_printing_quota.txt).

The campus shuttle runs a loop every 20 minutes from 7am to 11pm on weekdays.
Source: transit_shuttle.txt

The library is open until 2am during term.
Sources: housing_morrow_house_noise.txt, housing_calder_annexe_noise.txt, housing_tamsin_court_noise.txt, housing_fenwick_court_noise.txt, housing_old_brewhouse_noise.txt
```

**Criterion 3 — gate stops out-of-corpus questions:**

```
What is the capital of Mongolia?        best distance 0.864 → refused
How do I change the oil in a diesel engine?  best distance 0.911 → refused
Who won the 1994 World Cup?             best distance 0.874 → refused
What is the recommended dosage of ibuprofen? best distance 0.824 → refused
How do I write a for loop in Rust?      best distance 0.831 → refused
```

**Criterion 4 — each chunk reads as a complete thought (run 1, sample):**

```
[from admin_pass_fail_option.txt]
Any course outside your major can be taken pass/fail, and — the part nobody mentions —
you can declare it as late as week eight, after you've seen your midterm. A pass needs
a C- or better. Two per year, maximum eight across a degree.
```
No sentence cut at either end. Self-contained. ✓

**Criterion 5 — named source is the correct one (run 1):**

Every cited file was opened and confirmed to contain the answer given:
- admin_pass_fail_option.txt → "week eight" ✓
- admin_dining_dollars.txt → "disappears" ✓
- admin_printing_quota.txt → "$30" ✓
- transit_shuttle.txt → "20 minutes" ✓
- housing_*_noise.txt → "2am during term" ✓

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer | MET | All three runs returned the correct chunk for all 5 questions. The right document was always in the top-5 results — the corpus stores one fact per file, so a close distance means the right file. |
| 2 | Every answer names a source | MET | Every answer in all three runs cited a filename, either as "Source: filename" or inline as "(filename)". Both formats name the document, so all 5 of 5 passed each run. |
| 3 | Gate stops out-of-corpus questions | MET | Retrieval is deterministic and the gate is a fixed threshold comparison, so this is one pass: all 5 out-of-scope questions were refused at distances 0.82–0.91, well above the 0.6 cutoff. |
| 4 | Each chunk reads as a complete thought | MET | The paragraph chunker splits on blank lines, so every chunk begins and ends at a natural boundary. I read the retrieved chunks for all 5 questions across run 1 — none had a sentence cut at either end. |
| 5 | Named source file is the correct one | MET | I opened each cited file and confirmed the answer text is in it. The only slightly ambiguous case is the library question, where the answer appears in housing noise posts rather than a dedicated library file — but those files genuinely contain the fact, so the citation is correct. |

## Diagnoses

No misses across any of the three runs.

That result is honest, but the targets were conservative in two places:

**Criterion 1 (retrieved chunk contains the answer) — target: 4 of 5.** The `campus_life` corpus stores one fact per file with clear filenames, so the right document almost always ranks first. Getting 5 of 5 every run was not surprising given how the corpus is structured. I'd tighten this to 5 of 5 — it reflected uncertainty I had before seeing any results, and the results showed that uncertainty was unwarranted.

**Criterion 3 (gate stops out-of-corpus questions) — target: 4 of 5.** The gap between in-corpus distances (0.22–0.46) and out-of-corpus distances (0.82–0.91) is 0.36 wide. There is no ambiguous zone. Allowing one miss was unnecessary — I'd tighten this to 5 of 5 as well.

The other three criteria (source naming, complete chunks, correct source) were set at 5 of 5 or 4 of 5 and held. Those targets were appropriate.

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
