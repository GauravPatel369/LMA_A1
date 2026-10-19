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

I ran my experiments on the official text of Regulation (EU) 2016/679 (GDPR), which runs across 88 pages with 173 recitals and 99 articles.

While inspecting the raw text extracted with pypdf, I noticed that repeated running headers on every page (4.5.2016 L 119/n Official Journal of the European Union EN) and footnote citations caused false keyword matches. I removed these before creating chunks.

To avoid splitting legal rules across chunks, I wrote a structure-aware chunker using a sliding window of 220 words with a 40-word overlap, anchored to Recital and Article boundaries. This yielded 405 indexed chunks. In Section 5 of my notebook, I added self-tests to confirm that all 173 recitals and 99 articles were parsed contiguously before running embeddings.

Both pipelines used the exact same 405 chunks:
- Pipeline 1 (Baseline Retrieval): The query was prefixed with the required BGE instruction ("Represent this sentence for searching relevant passages: "), encoded with bge-small-en-v1.5, and queried against FAISS. The top 10 candidates were retrieved, and the top 3 were given to the generator.
- Pipeline 2 (Re-Ranked Retrieval): The same top 10 candidates from Stage 1 were paired with the query, scored with ms-marco-MiniLM-L-6-v2, sorted descending by score, and the top 3 were given to the generator.
- Generation: Both pipelines passed their top 3 chunks to an identical prompt using qwen/qwen3.8-27b on Groq with temperature set to 0, ensuring that any variation in answers came only from the retrieved text.

---

3. Rank Shifts and Answer Comparisons

Across my 14 test queries (12 failure probes and 2 control queries), Recall at 3 increased from 7/14 (50%) in the baseline to 9/14 (64.3%) after re-ranking. The Stage 1 ceiling (Recall at 10) was 12/14 (85.7%).

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

Out of the 5 queries with headroom (where the gold passage sat between ranks 4 and 10), re-ranking brought 3 into the top 3 (Q1, Q2, and Q3). 

Case Q1: Negation and Jargon Gap (Recital 71, Rank 4 to Rank 1)
When I asked "Can a bank refuse my loan application using only an algorithm with no human involved?", the baseline answer gave up. Because Recital 71 sat at rank 4, the model had only Recitals 18, 43, and 28 in context, and stated that the provided text did not contain enough information. Answer 2, with Recital 71 at rank 1, explained that automated credit refusal without human intervention is generally prohibited, and listed the three exceptions (statutory authorisation, contractual necessity, or explicit consent). The bi-encoder missed this passage because the query used everyday words ("bank loan", "algorithm"), whereas the regulation uses legal phrasing ("automated processing", "credit application"). The cross-encoder identified this connection through joint attention.

Case Q2: Keyword Trap and Missing Exceptions (Article 34, Rank 5 to Rank 2)
On Query 2 ("When must we tell the affected individuals themselves about a breach?"), the bi-encoder was distracted by the word "breach" and pulled in Article 33 (notifying the regulator within 72 hours) and general recitals. Answer 1 cited Recital 86 and stated the broad high-risk threshold, but missed the actual operative rule. Answer 2 cited Article 34(1) and explained the three statutory exemptions under Article 34(3) where notifying individuals is not required, such as when data is encrypted. In practice, missing these exemptions could lead an organisation to send unnecessary breach notifications when the law does not require them.

Case Q3: Buried Clause (Article 26, Rank 8 to Rank 1)
For Query 3 ("If two companies decide together how data is used, who is responsible?"), the baseline only surfaced Recitals 74 and 79, leading Answer 1 to state vaguely that responsibilities must be allocated clearly. The cross-encoder brought Article 26 up from rank 8 to rank 1. Answer 2 provided the specific legal requirement: joint controllers must put in place a transparent agreement determining who handles data subject access requests under Articles 13 and 14.

Case Q6: The Regression Case (Article 5, Rank 1 to Rank 4)
One result that surprised me was Query 6 ("How long are we allowed to keep customer records?"). Here, re-ranking made the answer worse. In the baseline, Article 5(1)(e) was at rank 1, and Answer 1 clearly explained the storage limitation principle along with research exemptions. The cross-encoder demoted Article 5 to rank 4 and promoted Recital 137 and Recital 39 instead. Answer 2 had to rely solely on Recital 39, losing the primary statutory reference. When I checked why this happened, Recitals 137 and 39 repeat phrases about time limits ("period for which", "time limits should be established", "erasure"). Article 5 expresses the rule more compactly as "storage limitation". The cross-encoder gave higher scores to the descriptive wording in the recitals, demoting the binding article. This showed that cross-encoders can sometimes prefer descriptive text over compact legal rules.

The Recall Ceiling on Queries 7 and 8
For Query 7 (Article 32) and Query 8 (Article 48), the bi-encoder failed to include the gold articles anywhere in the top 10. The cross-encoder could not help because it can only reorder the candidates it is given. If Stage 1 does not retrieve a document, the pipeline cannot recover it.

---

4. Latency Profiling

I measured the execution time of each stage across 10 runs (discarding the first cold-start run) on a standard CPU runtime. I kept local retrieval measurements separate from the remote Groq API call, because network variations on cloud APIs would hide the small millisecond differences in retrieval.

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

Re-ranking increased retrieval time by roughly three times (from 11.67 ms to 35.44 ms). However, an extra 24 ms is small when compared to the 450 to 640 ms taken by the LLM to generate text (roughly 5% of the total turnaround time). In practice, adding 24 ms to improve recall from 50% to 64% and retrieve missing legal exemptions is an acceptable trade-off.

---

5. Scaling: Why Not Cross-Encode the Entire Corpus?

One natural question is whether we can skip the bi-encoder entirely and score all passages with the cross-encoder from the start.

This comes down to computational scaling:
- A bi-encoder's work is pre-computed: Generating embeddings for all 405 chunks took 1.85 seconds once during indexing. At search time, only the user query is embedded (11.6 ms), and vector search scales logarithmically or in constant time.
- A cross-encoder cannot pre-compute document representations: Because the model requires both the query and the chunk tokens together, no document vector exists beforehand. Every search requires N full forward passes through the transformer, meaning computation scales linearly with the number of chunks.

I measured this in Section 14 of my notebook by scoring a probe query against all 405 chunks:
- Scoring all 405 chunks took 0.634 seconds (about 1.56 ms per pair).
- Re-ranking the top 10 candidates took 0.024 seconds (23.8 ms).
- Full-corpus cross-encoding was already 27 times slower on just 405 chunks.

Extrapolating this cost of 1.56 ms per pair shows why it does not scale:
- 10,000 chunks (about 50 policy documents): 1.56 ms x 10,000 = 15.6 seconds per query.
- 100,000 chunks (an enterprise document base): 1.56 ms x 100,000 = about 2.6 minutes per query.
- 1,000,000 chunks (a large legal database): 1.56 ms x 1,000,000 = about 26 minutes per query.

This confirms why two-stage retrieval is standard practice: the bi-encoder quickly narrows the search space to 10 candidates in milliseconds, and the cross-encoder spends its computation only on that shortlist.

---

6. Closing Observations

From these experiments, I took away three main practical points:
1. Re-ranking is valuable when queries involve negations or everyday phrasing, where vector similarity tends to leave important clauses between ranks 4 and 10.
2. The initial candidate window should be widened to 30 or 50 chunks, ideally combining dense retrieval with BM25 keyword search, so that the pipeline is not limited by the Stage 1 recall ceiling seen in Q7 and Q8.
3. System builders should watch for regressions like Query 6, where wordy recitals outscore concise statutory rules. Adding a lexical match or article booster can help keep primary principles at the top.
