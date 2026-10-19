# Baseline Retrieval vs. Cross-Encoder Re-Ranking

A two-stage RAG study on the **EU General Data Protection Regulation**
(Regulation (EU) 2016/679) — 88 pages, 173 recitals and 99 articles.

Both pipelines run over the **same chunked corpus**:

| | Pipeline 1 (baseline) | Pipeline 2 (re-ranked) |
|---|---|---|
| Retrieve | `bge-small-en-v1.5` → FAISS → top 10 | the *same* top 10 |
| Select | top 3 by cosine similarity | top 3 after cross-encoder re-scoring |
| Generate | Answer 1 | Answer 2 |

The prompt, the model and the decoding settings are identical. The retrieved
context is the only variable.

## Running it

**Google Colab** (CPU runtime is enough — a GPU only changes timings, not results; the numbers reported below were measured on an RTX 4050):

1. Open `RAG_Baseline_vs_Reranking_GDPR.ipynb` in Colab
2. Add your Groq API key:  sidebar icon -> **Add new secret** -> name it  `GROQ_API_KEY` -> enable notebook access
3. **Runtime -> Run all**

**Kaggle:** enable internet first (Account -> Phone Verification, then
Settings -> Internet), add the key under Add-ons -> Secrets with the same name,
then run. The notebook reads Colab Secrets, Kaggle Secrets, or a `GROQ_API_KEY`
environment variable, in that order.

**Locally (CUDA GPU optional):**

```bash
python -m venv .venv
.venv/Scripts/python -m pip install torch --index-url https://download.pytorch.org/whl/cu124
.venv/Scripts/python -m pip install "pypdf==6.20.0" "groq==0.33.0" \
    "sentence-transformers>=5.1" "faiss-cpu>=1.13" "numpy<2.3" nbconvert ipykernel
export GROQ_API_KEY=...        # Windows: $env:GROQ_API_KEY="..."
.venv/Scripts/python -m jupyter nbconvert --to notebook --execute \
    --output RAG_Baseline_vs_Reranking_GDPR_result.ipynb \
    RAG_Baseline_vs_Reranking_GDPR.ipynb
```

The notebook detects CUDA and reports the device it used; drop the `--index-url`
line for a CPU-only install and everything still runs. **Ranks and answers are
identical on CPU and GPU** - only wall-clock timings move, so the GPU is a
convenience and never part of the argument.

Get a free Groq key at [console.groq.com](https://console.groq.com). No paid API
is used anywhere: the embedder and cross-encoder are small local models (33M and
22M parameters), and generation runs on Groq's free tier.

The notebook downloads the GDPR PDF from EUR-Lex itself, so it runs end-to-end
with nothing but the key.

## What it produces

Written to `outputs/`:

| File | Contents |
|---|---|
| `chunks.json` | the 405 chunks with their article/recital metadata |
| `results.json` | per query: all 10 candidates with both rankings, both answers, timings |
| `summary.json` | Recall@3, latency table, scaling measurement, resolved library versions |
| `answers.md` | Answer 1 vs Answer 2 side by side, ready to quote |

## The self-test

Section 5 asserts its own preprocessing before anything is indexed — 20 checks
covering structural integrity (all 173 recitals and 99 articles parsed
contiguously), page-furniture removal, and that operative text landed in the
correct unit.

The last check is the one that matters: **Art 9(1) ("shall be prohibited") and
Art 9(2)(h) (the healthcare exception) must be in different chunks.** If a chunk
size ever let them merge, the headline failure mode would not exist and the study
would be measuring nothing. The notebook raises rather than reporting a result
that looks fine but is hollow.

Three preprocessing problems it defends against, each found by inspecting the
actual file rather than assumed:

1. All 88 pages carry the line `4.5.2016 L 119/n Official Journal of the European
   Union EN`. It has to be stripped **as a whole line**, not as a phrase — real
   article text also says "Official Journal of the European Union" (Art 45(8),
   Art 92, Art 99(1)), and a phrase-level strip silently corrupts those articles.
2. pypdf preserves the PDF's visual line breaks, so `"...shall be "` and
   `"prohibited."` arrive as separate lines. Without unwrapping, no sentence-level
   match succeeds.
3. `Article 29`, its title and its body are three separate lines. Merge them and
   the 16 single-paragraph articles (10, 16, 19, 29, 31, 44, 48, 50, 59, 63, 67,
   86, 87, 95, 96, 98) fold their body into the title line and drop out of the
   corpus entirely. Art 29 is the gold answer for query Q5.

Footnotes are rejected by a sequence guard: they restart numbering on every page
(`(1) OJ C 229, 31.7.2012, p. 90.`), so `(n)` only opens a recital when `n`
continues the 1..173 run.

## How each requirement is answered

| Assignment section | Where it is answered |
|---|---|
| 1. Setup & environment | notebook SS1-2; the PDF is validated by SHA-256 and page count before use |
| 2. Pipeline 1 (bi-encoder -> FAISS -> top 10 -> top 3 -> Answer 1) | notebook SS6-7, SS10 |
| 2. Pipeline 2 (same 10 -> cross-encoder -> top 3 -> Answer 2) | notebook SS7, SS10 - `retrieve_candidates` runs **once** and both pipelines consume its output |
| 3. >= 2 failure queries rescued by re-ranking | **3 rescues** - Q1, Q2, Q3 |
| 3. Keyword trap / distractor | Q2 (Art 34 vs Art 33 + Recitals 85/86) |
| 3. Exceptions & corner cases | Q9 (Art 9(1) prohibits, Art 9(2)(h) permits), Q10 (Art 17 carve-outs) |
| 3. Negation & constraints | Q1 (Art 22 negative right), Q11 (Art 29 role limit), Q12 (Art 2 scope) |
| 3. Vital chunk buried at 4-10, promoted into the top 3 | Q3 (#8 -> #1) |
| 4.1 Architecture comparison | notebook SS6, SS14 - measured, not only asserted |
| 4.2 Rank shift + Answer 1 vs Answer 2 | notebook SS11-12, `outputs/answers.md` |
| 4.3 Latency profiling | notebook SS13 |
| 4.4 Scaling question | notebook SS14 - the full-corpus cross-encode is measured directly |

## Query set

Fourteen queries, screened against the baseline **before** being adopted
(`DIAGNOSTIC_query_mining.ipynb` mines a 26-candidate pool). The screen sorts a
query by where the gold chunk lands, because that decides what re-ranking can
possibly do:

| band | meaning | in this set |
|---|---|---|
| 1-3 | baseline already succeeds - no headroom | 7 |
| **4-10** | **a rescue is possible** - in the pool, outside the context window | **5** |
| >10 | outside the candidate window - the cross-encoder never sees it | 2 |

**All five headroom queries are included, not only the three that worked.**
Keeping only the winners would make the re-ranker look better than it is: the
honest ratio is **3 rescued, 2 not**.

Those outside the window are *not* unretrievable - sweeping the cutoff over the
pool gives recall@3 = 15, @10 = 20, @20 = 23, @50 = 25, **@100 = 26 of 26**. The
ceiling is the *candidate window*, not the bi-encoder's recall, so **re-ranking
repairs ordering, never recall**. The assignment fixes `TOP_K = 10`, so that
stays the graded configuration.

Gold is any chunk containing the operative text, **article or recital**, and
every probe is pinned by an assertion to the exact units it may resolve to - a
loose fragment cannot manufacture a hit.

| | Category | Gold | Base | Re-ranked | Effect |
|---|---|---|---|---|---|
| Q1 | negation / jargon gap | Art 22 / Rec 71 | #4 | **#1** | **rescued** |
| Q2 | keyword trap / distractor | Art 34 | #5 | **#2** | **rescued** |
| Q3 | buried clause / role limit | Art 26 | #8 | **#1** | **rescued** |
| Q4 | constraint / buried clause | Art 25 | #4 | #7 | not rescued |
| Q5 | role limit / contract | Art 28 | #6 | #7 | not rescued |
| Q6 | regression probe | Art 5(1)(e) | #1 | #4 | **regressed** |
| Q7 | recall ceiling | Art 32 | >10 | >10 | unreachable |
| Q8 | recall ceiling | Art 48 | >10 | >10 | unreachable |
| Q9 | exception / corner case | Art 9(2)(h) | #3 | #2 | both OK |
| Q10 | exception / corner case | Art 17 / Rec 65 | #1 | #1 | both OK |
| Q11 | negation / role limit | Art 29 | #1 | #1 | both OK |
| Q12 | negation / scope | Art 2 / Rec 18 | #2 | #2 | both OK |
| C1 | control | Art 83 | #1 | #1 | both OK |
| C2 | control | Art 4 | #1 | #1 | both OK |

## Results

| metric | baseline | re-ranked | delta |
|---|---|---|---|
| **Recall@3** (all 14) | 7/14 (50%) | **9/14 (64%)** | +2 |
| Recall@3 (failure probes) | 5/12 (42%) | 7/12 (58%) | +2 |
| Recall@3 (controls) | 2/2 | 2/2 | 0 |
| Recall@10 (stage-1 ceiling) | 12/14 (86%) | 12/14 (86%) | shared |
| **MRR** | 0.487 | **0.574** | +0.086 |
| Answer cites the gold unit | 7/14 (50%) | **10/14 (71%)** | **+3** |
| Answer contains the key fact | 13/14 (93%) | 12/14 (86%) | -1 |

Every metric is over **all 14 queries**; a gold chunk that never entered the top
10 counts as a miss (reciprocal rank 0) rather than being dropped from the
denominator.

**The citation row is the interesting one.** Re-ranking lifts correct
attribution from 50% to 71% while the key-fact rate barely moves. The LLM
usually *states* the right rule either way - it can paraphrase it from a recital
- but without the binding article in context it attributes the rule to
non-binding recitals. In a compliance setting that distinction is the whole
point.

**The single key-fact loss is Q6**, the one rank regression: gold fell #1 -> #4
and the answer lost both its citation and its key fact. The rank damage
propagates straight through to the generated text, which is the clearest
evidence in the run that retrieval order decides answer quality.

## What the rank shift bought (section 4.2)

- **Q2** - Answer 1 cites only **Recital 86**, which is not binding, and omits
  Art 34(3)'s three exceptions entirely. Answer 2 cites Art 34(1) and lists them.
- **Q3** - Answer 1 answers from Recitals 74/79; Answer 2 cites **Article 26**
  and the joint-controller arrangement. Right answer, wrong source of authority.

Both answers for every query: `outputs/answers.md`.

## Measured latency (section 4.3)

RTX 4050 Laptop GPU, median of 10 runs, warm-up discarded:

| Stage | GPU (RTX 4050) | CPU |
|---|---|---|
| query embedding (bi-encoder) | 45.3 ms | 32.8 ms |
| FAISS search (top-10) | 0.1 ms | 0.1 ms |
| cross-encoder rerank (10 pairs) | 42.7 ms | 904.4 ms |
| **Pipeline 1 retrieval** | **45.5 ms** | **32.9 ms** |
| **Pipeline 2 retrieval** | **88.1 ms** | **937.3 ms** |
| one-off index build (405 chunks) | 3.5 s | 61.6 s |

Query embedding does not improve on the GPU - it is marginally slower - because
one short string is dominated by tokenisation and kernel-launch overhead, not
arithmetic. The GPU pays off only where there is a batch to fill: the index
build drops 18x. Re-ranking overhead falls from 27.5x baseline retrieval to
0.94x, so **the cost of re-ranking is hardware-dependent, but the reason you
cannot cross-encode the whole corpus is not.** Scoring all 405 chunks still
costs 2.0 ms/pair on the GPU - about 2,000 s per query at a million chunks -
because a cross-encoder score exists only for a specific (query, document) pair
and cannot be precomputed, so the work grows linearly with the corpus on any
hardware.

## Notes

- `temperature=0` throughout.
- The Groq free tier allows roughly 8,000 tokens/minute and each call costs about
  1,400, so the notebook paces requests and backs off on 429. A full run takes a
  few minutes, most of it waiting on that limit rather than computing.
- Runs on CPU or CUDA; the notebook prints which. The reported run used an
  RTX 4050. The GPU changes timings only - every rank, answer and Recall@3
  figure above is identical on CPU.
