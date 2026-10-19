RAG Assignment

Topic: Baseline Retrieval vs. Cross-Encoder Re-Ranking on the GDPR
Student Name: Gaurav Patel
Roll Number: 2025201065
Course: Large Multimodal Models and Applied NLP
Document Analysed: Regulation (EU) 2016/679 (GDPR, 88 pages, 405 chunks)
Models Used: bge-small-en-v1.5 (Bi-Encoder), ms-marco-MiniLM-L-6-v2 (Cross-Encoder), qwen/qwen3.8-27b (via Groq API)
Repository: https://github.com/GauravPatel369/LMA_A1

---

1. Architectural Comparison: Bi-Encoder vs. Cross-Encoder

A bi-encoder (bge-small-en-v1.5) embeds queries and documents independently into separate dense vectors. Pre-computing document vectors enables fast sub-millisecond similarity search in FAISS (0.07 ms in my runs), but vector compression loses subtle nuances like negations and specific qualifiers.

In contrast, a cross-encoder (ms-marco-MiniLM-L-6-v2) processes query-passage pairs jointly with full token-to-token cross-attention. While this accurately captures negations and legal exceptions, evaluating each pair requires a full forward pass (2.38 ms per pair), making it practical only for re-ranking a small candidate shortlist.

---

2. Dataset Processing and Experimental Setup

I evaluated both pipelines on Regulation (EU) 2016/679 (GDPR), which spans 88 pages with 173 recitals and 99 articles.

Using pypdf, I extracted text and stripped repeating running headers and footnote citations to avoid false keyword matches. I built a structure-aware chunker using a 220-word sliding window with a 40-word overlap anchored to recital and article boundaries, producing 405 indexed chunks. Pre-indexing self-tests in Section 5 of my notebook confirmed that all recitals and articles were parsed contiguously.

Both pipelines run over these identical 405 chunks:
- Pipeline 1 (Baseline Retrieval): The query is prefixed with the BGE instruction, encoded with bge-small-en-v1.5, and queried against FAISS to retrieve the top 10 candidates. The top 3 are sent to the generator.
- Pipeline 2 (Re-Ranked Retrieval): The same top 10 candidates are scored with ms-marco-MiniLM-L-6-v2, sorted descending, and the top 3 are sent to the generator.
- Generation: Both pipelines supply their top 3 chunks to an identical prompt using qwen/qwen3.8-27b on Groq at temperature 0, isolating retrieved context as the sole variable.

---

3. Rank Shifts and Answer Comparisons

Across 14 test queries (12 failure probes and 2 controls), Recall at 3 improved from 7/14 (50%) in the baseline to 9/14 (64.3%) after re-ranking, against a Stage 1 ceiling (Recall at 10) of 12/14 (85.7%).

Query Rank Shift Summary:

| Query | Target Provision | Baseline Rank | Re-Ranked Rank | Net Result |
| :--- | :--- | :---: | :---: | :--- |
| Q1 Automated loan refusal (negation/jargon) | Recital 71 | Rank 4 | Rank 1 | Rescued into top 3 |
| Q2 Tell individuals about breach (keyword trap) | Article 34 | Rank 5 | Rank 2 | Rescued into top 3 |
| Q3 Two companies decide data use jointly | Article 26 | Rank 8 | Rank 1 | Rescued into top 3 |
| Q4 Privacy built into design | Article 25 | Rank 4 | Rank 7 | Remained outside top 3 |
| Q5 Contract terms with data processor | Article 28 | Rank 6 | Rank 7 | Remained outside top 3 |
| Q6 Duration allowed to keep records | Article 5(1)(e) | Rank 1 | Rank 4 | Regressed out of top 3 |
| Q7 Technical measures against database theft | Article 32 | Not in top 10 | Not in top 10 | Missed by Stage 1 |
| Q8 Foreign court demanding European data | Article 48 | Not in top 10 | Not in top 10 | Missed by Stage 1 |
| C1 Maximum administrative fine (Control) | Article 83 | Rank 1 | Rank 1 | Retained at Rank 1 |
| C2 Definition of personal data (Control) | Article 4 | Rank 1 | Rank 1 | Retained at Rank 1 |

Re-ranking rescued 3 out of the 5 queries that had headroom between ranks 4 and 10 (Q1, Q2, and Q3).

Case Q1: Negation and Jargon Gap (Recital 71, Rank 4 to Rank 1)
When asking about automated loan refusal without human intervention, the baseline missed the relevant passage because the query used conversational phrasing ("bank loan", "algorithm") while the text uses formal legal terminology ("credit application", "automated processing"). Because Recital 71 sat at rank 4, Answer 1 stated there was insufficient information. The cross-encoder promoted Recital 71 to rank 1 via joint attention, allowing Answer 2 to accurately state the general prohibition and its three statutory exceptions.

Case Q2: Keyword Trap and Missing Exceptions (Article 34, Rank 5 to Rank 2)
On Query 2 regarding when individuals must be informed of a data breach, the bi-encoder was distracted by the keyword "breach" and retrieved Article 33 (notifying supervisory authorities) and generic recitals. Answer 1 cited non-binding Recital 86 and omitted the operative rule. The cross-encoder promoted Article 34 from rank 5 to rank 2. Answer 2 correctly identified Article 34(1) and detailed the three statutory exemptions under Article 34(3), including prior encryption.

Case Q3: Buried Clause (Article 26, Rank 8 to Rank 1)
For Query 3 on joint data responsibility, the baseline retrieved high-level recitals, leaving Answer 1 vague. The cross-encoder elevated Article 26 from rank 8 to rank 1. Answer 2 supplied the binding statutory obligation: joint controllers must enter into a transparent arrangement determining who responds to data subject access requests under Articles 13 and 14.

Case Q6: The Regression Case (Article 5, Rank 1 to Rank 4)
In Query 6 on record retention limits, re-ranking degraded performance. Baseline retrieval placed Article 5(1)(e) at rank 1, producing an accurate summary of storage limitation. The cross-encoder demoted Article 5 to rank 4 in favour of Recitals 137 and 39, which repeated descriptive phrases about time limits. Answer 2 lost the primary statutory citation. This shows that cross-encoders can sometimes favour descriptive phrasing in recitals over concise operative articles.

The Recall Ceiling on Queries 7 and 8
For Query 7 (Article 32) and Query 8 (Article 48), the target provisions were absent from the top 10 candidates retrieved by the bi-encoder. Because a cross-encoder only reorders candidates provided by Stage 1, omissions at the initial retrieval stage cannot be recovered.

---

4. Latency Profiling

I measured execution latencies across 10 evaluation runs on a standard CPU runtime, separating local retrieval timings from the remote Groq API generation call.

Stage Latency Summary:

| Stage | Median Latency |
| :--- | :---: |
| Stage 1: Query embedding (bge-small-en-v1.5) | 11.60 ms |
| Stage 1: FAISS vector search (top 10 over 405 chunks) | 0.07 ms |
| Pipeline 1 Total (Baseline Retrieval) | 11.67 ms |
| Stage 2: Cross-encoder re-ranking (10 candidate pairs) | 23.77 ms |
| (Average cost per candidate pair) | 2.38 ms / pair |
| Pipeline 2 Total (Two-Stage Retrieval) | 35.44 ms |
| Generation: LLM Answer 1 (Groq API) | 452.62 ms |
| Generation: LLM Answer 2 (Groq API) | 640.72 ms |

Re-ranking increased retrieval latency from 11.67 ms to 35.44 ms. However, this extra 24 ms represents only about 5% of total turnaround time, which is dominated by LLM generation (~500 ms). Adding 24 ms to raise recall from 50% to 64.3% and surface key statutory exceptions is an effective engineering trade-off.

---

5. Scaling: Why Not Cross-Encode the Entire Corpus?

Skipping the bi-encoder to cross-encode an entire corpus is unviable due to computational scaling:
- Bi-encoder indexing is pre-computed: Embedding all 405 chunks took 1.85 seconds once. At query time, only the user query is encoded (11.6 ms), and vector search scales sub-linearly.
- Cross-encoders require joint inputs: Because the query and passage must be processed together, no document vectors exist in advance. Every search requires N full transformer passes, scaling linearly with corpus size.

In Section 14 of my notebook, scoring a query across all 405 chunks took 0.634 seconds (~1.56 ms per pair), already 27 times slower than re-ranking top 10 candidates (0.024 seconds). Extrapolating this cost illustrates the scaling bottleneck:
- 10,000 chunks: 15.6 seconds per query.
- 100,000 chunks: 2.6 minutes per query.
- 1,000,000 chunks: 26 minutes per query.

Two-stage retrieval resolves this by using fast bi-encoder vector search to filter thousands of chunks down to 10 candidates, reserving cross-attention compute exclusively for the shortlist.

---

6. Closing Observations

Three main takeaways emerge from this study:
1. Re-ranking is particularly effective for queries involving negations, legal jargon, and buried clauses, lifting recall from 50% to 64.3% and binding citations from 50% to 71.4%.
2. Initial candidate pools should be widened (e.g. top 30 to 50) and hybridised with BM25 keyword matching to prevent Stage 1 recall ceilings as observed in Q7 and Q8.
3. System designers must monitor for regressions like Q6, where descriptive recitals outscore concise statutory rules. Lexical matching or article boosters can help preserve binding provisions.
