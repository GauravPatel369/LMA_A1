# Report outline

The brief is explicit: *"The report should not be AI-generated... in your own
words. We want to evaluate your actual understanding."*

So this file contains **no prose for you to copy** — only the structure, the
questions each section has to answer, and where in `outputs/` each number lives.
Write the sentences yourself. If a concept here is not solid when you get to it,
ask and work through it until you could explain it to someone else; that is the
thing being graded.

Run the notebook first. Every number below comes from `outputs/summary.json` or
`outputs/results.json`.

---

## 1. Architecture comparison

Explain the mechanical difference, not the definitions:

- A **bi-encoder** embeds the query and each document *separately*. A document's
  vector never sees the query, which is exactly why it can be computed once and
  stored in an index. Scoring is then one cheap dot product.
- A **cross-encoder** concatenates query and document and runs self-attention
  across the combined token sequence, so every query token can attend to every
  document token. The score exists only for that specific pair.

Then answer the "so what": name a concrete thing the cross-encoder can see that a
bi-encoder cannot. Your Q2 and Q5 results are about negation — think about what a
single fixed vector has to do to encode "shall not" versus "shall", and why joint
attention over the pair is a different problem.

Keep this short. Two or three paragraphs.

---

## 2. Qualitative and rank-shift analysis

**The table.** From `summary.json` → `rank_shifts`, or the printed table in
section 11 of the notebook. Columns: query, category, gold article, baseline
rank, re-ranked rank, effect.

**Recall@3.** From `summary.json` → `recall_at_3`. Report all three rows
(overall, failure probes, controls) — the controls row is the one that shows you
measured rather than curated.

**The answer comparison.** `outputs/answers.md` has Answer 1 and Answer 2 side
by side. Pick your two or three strongest cases and quote them. For each, say
precisely which of these happened:

- a **hallucination** — the answer asserts something absent from the context
- an **omitted clause** — the answer is incomplete but not wrong
- a **misstated rule** — the answer is confidently wrong about what the law says

These are different failures and the brief asks you to distinguish them. Name
which one you actually observed; do not reach for "hallucination" by default.

**Something to look at carefully on Q1.** The Art 9 chunk holding the 9(1)
prohibition is 220 words and also contains the start of 9(2) and sub-points
(a)–(d). So if the baseline retrieves it, the model can see that *exceptions
exist* and may cite 9(2)(a) explicit consent instead of 9(2)(h) treatment. If
that is what happens, it is a **misstated rule**, not a flat refusal — and a more
interesting finding, because consent is the wrong legal basis for treating a
patient who cannot give it. Check what your run actually produced and describe
that, not what you expected.

**If re-ranking hurt a query, report it.** Say which one and by how much. A
report showing uniform wins reads as curated; one honest regression is worth more
than six clean victories.

---

## 3. Latency profiling

Table from `summary.json` → `latency_ms`. Rows: query embedding, FAISS search,
cross-encoder re-rank (10 pairs), Pipeline 1 retrieval total, Pipeline 2
retrieval total, LLM generation.

State the method: median of 10 runs, warm-up call discarded, model load time
excluded, and **stages reported separately**. Say why separately — the LLM call
runs on Groq's hardware and its network variance is far larger than the
cross-encoder's cost, so a single end-to-end number would hide the one thing this
section is measuring.

State the hardware (Colab CPU) and the corpus size (405 chunks). Latency claims
without those are meaningless.

---

## 4. Scaling question

Why not cross-encode the entire corpus from the start?

Section 14 of the notebook measures this instead of asserting it: it scores the
query against all 405 chunks and reports the per-pair cost. Use your own numbers
from `summary.json` → `scaling`.

The argument to make in your own words:

- The bi-encoder's document work is **amortised** — embed once at index time,
  then every query is a cheap similarity search over precomputed vectors.
- The cross-encoder's work **cannot be amortised at all**, because joint
  attention means no per-document representation exists to cache. Every query
  costs N forward passes.
- Therefore cost is O(N) *per query*, and the printed extrapolation shows what
  that means at 10k and 1M chunks.
- Conclude with why the two-stage design follows: the bi-encoder reduces N to a
  constant (10), and the cross-encoder spends its expensive joint attention only
  on that shortlist.

Then connect it back to your own data: your C2 control (where re-ranking may have
regressed) is evidence that this extra cost is not always repaid. That link
between section 4 and your measured results is worth making explicitly.

---

## Checklist

- [ ] Notebook runs top to bottom on Colab with no errors
- [ ] Self-test passes (section 5)
- [ ] All four sections written in your own words
- [ ] Rank-shift table included
- [ ] At least two failure queries analysed with quoted answers
- [ ] Latency table with method and hardware stated
- [ ] Any regression reported honestly
- [ ] Groq key rotated before the repo goes public
- [ ] `api.txt` not committed (`.gitignore` covers it — verify with `git status`)
