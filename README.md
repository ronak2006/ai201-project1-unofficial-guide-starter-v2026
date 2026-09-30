# The Unofficial Guide

campus_life

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

This system answers questions about campus life using a RAG pipeline. The corpus is a collection of short 
student-written guides covering dining halls, housing, courses, and administrative policies. When asked a question, the system finds 
the most relevant document and generates an answer grounded in that text. It also does not answer questions not covered by the corpus.

## Chunking Strategy

**Chunk size:** 600 characters
**Overlap:** 0

Each campus_lif document covers exactly one topic with a clear heading like "On the ____" and runs 5–6 sentences. The longest document is 549 characters. Because every document already fits in one chunk and covers a single idea, splitting them further would only create chunks that lose context. One document equals one chunk. Overlap is zero because there are never two consecutive chunks from the same document to bridge.

## Sample Chunks

**Chunk 1** — source:admin_add_drop_deadline.txt#0 `` — produced by:chunker.py::split_documents ``

```
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** — source: course_biol_160.txt#0 `` — produced by:chunker.py::split_documents ``

```
BIOL 160 Cell Biology

I lived here my sophomore year. Format is lecture three times a week with a weekly lab. Assessment: four unit tests and a cumulative final. Not curved.

Expect 9 to 11 hours a week, the heaviest first-year course by reputation.

The one piece of advice: the unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.

```

**Chunk 3** — source: course_hist_118_workload.txt#0`` — produced by: chunker.py::split_documents``

```
Workload for HIST 118 Modern World History

People keep asking so: a lot of reading, about 120 pages a week, but no problem sets. That's real time, not optimistic time.

It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```

**Chunk 4** — source: dining_pellew_dining_hall_followup.txt#0`` — produced by: chunker.py::split_documents``

```
Re: Pellew Dining Hall

Adding to what people have said about Pellew Dining Hall. The wait figure of 12 to 18 minutes at peak matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely.

Also worth saying: the furthest hall from anywhere, next to the athletics centre. Nobody tells you this at orientation.
```

**Chunk 5** — source: housing_innisfree_hall.txt#0`` — produced by:chunker.py::split_documents ``

```
Innisfree Hall — what it's actually like

Transferred in last year, so take this with a grain of salt. Built 1991, renovated 2022. Rooms are doubles arranged as pairs sharing one bathroom between two rooms.

The good: the shared-bathroom-between-two-rooms arrangement is the best compromise on campus.

The bad: no air conditioning, which matters for the first three weeks of September.

Laundry costs $1.75 wash, $1.75 dry, app-based. On noise: moderate; the building is L-shaped and the short wing is much quieter.
```

## Sample Answer

**Question:** How much does laundry cost in Innisfree Hall?

**Answer:**

```
In Innisfree Hall, laundry costs $1.75 for a wash and $1.75 for a dry.

This information came from `housing_innisfree_hall.txt` and `housing_innisfree_hall_laundry.txt`.

Sources retrieved: housing_aldridge_hall.txt, housing_aldridge_hall_laundry.txt, housing_calder_annexe.txt, housing_innisfree_hall.txt, housing_innisfree_hall_laundry.txt
```

**My relevance cutoff:** 0.6

In-corpus questions scored between 0.205 and 0.289. Out-of-scope questions scored between 0.825 and 0.934. The gap between the two groups is large and clean, so 0.6 sits comfortably in the middle.

| Question | In corpus? | Best distance |
|---|---|---|
| What are the wait times at Kestrel Commons during the lunch rush? | Yes | 0.209 |
| How much does laundry cost in Innisfree Hall? | Yes | 0.205 |
| Can juniors and seniors pick housing before sophomores in the lottery? | Yes | 0.222 |
| Until what week can I switch a course to pass/fail? | Yes | 0.257 |
| How many credit hours are required to graduate? | Yes | 0.289 |
| What is the capital of Mongolia? | No | 0.825 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.844 |
| Who won the 1994 World Cup? | No | 0.886 |
| How do I write a for loop in Rust? | No | 0.896 |
| How do I change the oil in a diesel engine? | No | 0.934 |

## How I Used AI


**1.** I asked Claude to write the chunking function. It gave me a version with sentence-boundary splitting for long documents. Since all  
  my documents were under 600 characters, I simplified it to just make one chunk per document. 

**2.**  2. I asked Claude to check my chunking strategy. It confirmed that 1 doc = 1 chunk was the right call for short single-topic documents,and explained why overlap didn't matter in my case.



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
| 4. No chunk spans two topic documents | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Answer contains specific figure from source | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

Produced by `run_eval.py::main`. Full run log: `results/run_2026-09-30_1937_before.md`.

### Real output — Criterion 1 (retrieved chunk contains the answer)

From `run_eval.py::main`, `store.py::search` — run 1:

- **What are the wait times at Kestrel Commons during the lunch rush?**
  Best distance: 0.2091 · Sources: `dining_kestrel_commons.txt`, `dining_kestrel_commons_followup.txt` (and 3 others)
  Answer: *"Based on the documents, the wait times at Kestrel Commons are 20 to 25 minutes between 12:15 and 1:00"*
- **How much does laundry cost in Innisfree Hall?**
  Best distance: 0.2052 · Sources: `housing_innisfree_hall_laundry.txt`, `housing_innisfree_hall.txt` (and 3 others)
  Answer: *"In Innisfree Hall, laundry costs $1.75 for a wash and $1.75 for a dry."*
- **How many hours a week should I expect to spend on CS 210 outside of class?**
  Best distance: 0.2490 · Sources: `course_cs_210_workload.txt` (and 4 others)
  Answer: *"You should expect to spend 8 to 10 hours a week outside class on CS 210."*
- **Until what week can I switch a course to pass/fail?**
  Best distance: 0.2568 · Sources: `admin_pass_fail_option.txt` (and 4 others)
  Answer: *"You can declare a course pass/fail as late as week eight."*
- **How many credit hours are required to graduate?**
  Best distance: 0.2886 · Sources: `admin_graduation_requirements.txt` (and 4 others)
  Answer: *"To graduate, 120 credit hours are required."*

### Real output — Criterion 2 (every answer names a source)

From `run_eval.py::main` — run 1. Every answer above names at least one source inline. Examples:
- *"Sources: `housing_innisfree_hall_laundry.txt` and `housing_innisfree_hall.txt`"*
- *"Source: `course_cs_210_workload.txt`"*
- *"Source: admin_pass_fail_option.txt"*

### Real output — Criterion 3 (gate stops out-of-corpus questions)

From `run_eval.py::check_out_of_scope`, cutoff 0.6. Refused 5 of 5:

| Out-of-scope question | Best distance | Gate |
|---|---|---|
| What is the capital of Mongolia? | 0.825 | refused |
| How do I change the oil in a diesel engine? | 0.934 | refused |
| Who won the 1994 World Cup? | 0.886 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.844 | refused |
| How do I write a for loop in Rust? | 0.896 | refused |

### Real output — Criterion 4 (no chunk spans two topic documents)

Single-document chunking strategy (`chunker.py::split_documents`) ensures each chunk is one complete document. Retrieved sources never mix content from two different topic files — confirmed by checking source lists above: each document name in the retrieval results corresponds to exactly one topic (e.g. `dining_kestrel_commons.txt`, `housing_innisfree_hall_laundry.txt`).

### Real output — Criterion 5 (answer contains specific figure)

From `run_eval.py::main` — run 1. Specific figures present in every answer:
- Kestrel Commons: *"20 to 25 minutes between 12:15 and 1:00"*
- Innisfree Hall laundry: *"$1.75 for a wash and $1.75 for a dry"*
- CS 210 workload: *"8 to 10 hours a week"*
- Pass/fail deadline: *"week eight"*
- Graduation requirement: *"120 credit hours"*

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer | MET | Every question's top results included the source doc with the answer. I checked that the answer text (e.g. "20 to 25 minutes", "$1.75", "8 to 10 hours") appeared in the named retrieved file, not just that the gate passed. 5/5 in all three runs against a 4/5 target. |
| 2 | Every answer names a source | MET | Read every answer across all three runs — each one names at least one source file inline. Source citation is in the generation prompt, so it held every time. 5/5 in all three runs. |
| 3 | Gate stops out-of-corpus questions | MET | All five out-of-scope questions had distances > 0.6 (range 0.825–0.934) and were refused. The gap between in-scope distances (0.21–0.29) and OOS distances (0.83–0.93) is large, so the cutoff is unambiguous. |
| 4 | No chunk spans two topic documents | MET | Single-document chunking strategy means each chunk is exactly one source file. Retrieved sources always have one topic per file name — no bleed-over possible by design. |
| 5 | Answer contains specific figure from source | MET | Every answer across all three runs contained an exact figure from the source (numbers, dollar amounts, time ranges). The corpus is figure-dense and retrieval landed the right chunk each time. 5/5 in all three runs against a 4/5 target. |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

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
