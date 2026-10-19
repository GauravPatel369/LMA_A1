# Baseline Retrieval vs. Cross-Encoder Re-Ranking

A two-stage RAG study on the EU General Data Protection Regulation (Regulation (EU) 2016/679) covering 88 pages, 173 recitals, 99 articles, and 405 chunks.

Both pipelines run over the exact same chunked corpus:

| Stage | Pipeline 1 (Baseline) | Pipeline 2 (Re-Ranked) |
|---|---|---|
| Retrieve | bge-small-en-v1.5 -> FAISS -> top 10 | Same top 10 from Stage 1 |
| Select | Top 3 by cosine similarity | Top 3 after ms-marco-MiniLM-L-6-v2 re-ranking |
| Generate | Answer 1 (qwen/qwen3.8-27b on Groq) | Answer 2 (qwen/qwen3.8-27b on Groq) |

The prompt, model, and generation settings (temperature = 0) are identical. Retrieved context is the only variable.

## Running the Notebook

- **Google Colab:** Open `RAG_Baseline_vs_Reranking_GDPR.ipynb`, add your `GROQ_API_KEY` under Secrets, and select **Runtime -> Run all**.
- **Kaggle:** Enable Internet (Account -> Settings), add `GROQ_API_KEY` under Add-ons -> Secrets, and run all.
- **Local:**
  ```bash
  python -m venv .venv
  # Windows: .venv\Scripts\activate | Linux/macOS: source .venv/bin/activate
  pip install torch sentence-transformers faiss-cpu pypdf groq numpy jupyter
  export GROQ_API_KEY="..."  # Windows PowerShell: $env:GROQ_API_KEY="..."
  python -m jupyter nbconvert --to notebook --execute RAG_Baseline_vs_Reranking_GDPR.ipynb
  ```

Free Groq API keys are available at [console.groq.com](https://console.groq.com). The local embedding and re-ranking models run efficiently on CPU or GPU with identical ranking results.

## Produced Outputs

Generated files in `outputs/`:
- `chunks.json`: The 405 indexed GDPR chunks with recital/article metadata.
- `results.json`: Full rankings, scores, generated answers, and latencies for all 14 queries.
- `summary.json`: Recall, MRR, latency benchmarks, and scaling measurements.
- `answers.md`: Side-by-side comparison of Answer 1 vs Answer 2.
- `report/REPORT.md` and `report/REPORT.docx`: Clean academic report (Times New Roman, pure black, 4 pages).

## Preprocessing and Integrity Checks

Section 5 of the notebook runs self-tests before indexing:
- Removes running page headers ("4.5.2016 L 119/n Official Journal...") and footnote numbering.
- Uses structure-aware chunking (220 words with 40-word overlap) aligned to Article and Recital boundaries.
- Asserts contiguous parsing of all 173 recitals and 99 articles.
- Verifies that Article 9(1) (general prohibition) and Article 9(2)(h) (healthcare exemption) land in separate chunks to properly test retrieval nuance.

## Query Set and Results

The benchmark evaluates 14 queries: 12 diagnostic probes (negations, jargon gaps, keyword traps, regressions) and 2 control queries.

### Summary Metrics

| Metric | Pipeline 1 (Baseline) | Pipeline 2 (Re-Ranked) | Net Change |
|---|---|---|---|
| Recall@3 (All 14) | 7/14 (50.0%) | 9/14 (64.3%) | +2 rescued |
| Recall@3 (Failure Probes) | 5/12 (41.7%) | 7/12 (58.3%) | +2 rescued |
| Recall@3 (Controls) | 2/2 (100%) | 2/2 (100%) | Maintained |
| Recall@10 (Stage 1 Ceiling) | 12/14 (85.7%) | 12/14 (85.7%) | Shared |
| Mean Reciprocal Rank (MRR) | 0.487 | 0.574 | +0.087 |
| Cites Gold Provision | 7/14 (50.0%) | 10/14 (71.4%) | +3 |

### Query Rank Shifts

| Query | Target Provision | Baseline Rank | Re-Ranked Rank | Net Result |
|---|---|---|---|---|
| Q1 Automated loan refusal | Recital 71 | Rank 4 | Rank 1 | Rescued into top 3 |
| Q2 Tell individuals about breach | Article 34 | Rank 5 | Rank 2 | Rescued into top 3 |
| Q3 Two companies decide data use | Article 26 | Rank 8 | Rank 1 | Rescued into top 3 |
| Q4 Privacy built into design | Article 25 | Rank 4 | Rank 7 | Outside top 3 |
| Q5 Contract terms with processor | Article 28 | Rank 6 | Rank 7 | Outside top 3 |
| Q6 Allowed storage duration | Article 5(1)(e) | Rank 1 | Rank 4 | Regressed |
| Q7 Technical security measures | Article 32 | >10 | >10 | Missed by Stage 1 |
| Q8 Foreign court demanding data | Article 48 | >10 | >10 | Missed by Stage 1 |
| C1 Maximum administrative fine | Article 83 | Rank 1 | Rank 1 | Retained |
| C2 Definition of personal data | Article 4 | Rank 1 | Rank 1 | Retained |

### Key Findings
1. **Re-Ranking Rescues Nuance:** In Q1 (everyday words vs legal terms), Q2 (distractor overlap), and Q3 (buried clause), cross-attention promoted the relevant legal rule into the top 3.
2. **Citation Authority:** Re-ranking increased operative legal citations from 50% to 71.4%. The baseline frequently cited non-binding recitals instead of operative articles.
3. **Stage 1 Recall Ceiling:** For Q7 and Q8, neither target was in the initial top 10. Re-ranking cannot rescue passages missing from Stage 1.
4. **Regression Risk (Q6):** The cross-encoder scored wordy recital descriptions higher than the concise operative rule in Article 5(1)(e), causing a rank drop.

## Measured Latency and Scaling

Measured median latencies across 10 evaluation runs:

| Stage | CPU Runtime | RTX 4050 GPU |
|---|---|---|
| Query Embedding (bge-small-en-v1.5) | 11.60 ms | 45.30 ms |
| FAISS Search (top 10 over 405 chunks) | 0.07 ms | 0.10 ms |
| Pipeline 1 Total Retrieval | 11.67 ms | 45.40 ms |
| Cross-Encoder Re-Ranking (10 pairs) | 23.77 ms | 42.70 ms |
| Pipeline 2 Total Retrieval | 35.44 ms | 88.10 ms |
| LLM Generation (Groq API) | 450 - 640 ms | 450 - 640 ms |

### Why Full-Corpus Cross-Encoding Does Not Scale
Scoring all 405 chunks with the cross-encoder took 0.634 seconds (~1.56 ms per pair). Because cross-encoders require joint query-document input, representations cannot be pre-computed:
- 10,000 chunks: 15.6 seconds per query.
- 100,000 chunks: 2.6 minutes per query.
- 1,000,000 chunks: 26 minutes per query.

Two-stage retrieval combines the sub-millisecond scaling of bi-encoders with the precision of cross-encoders on a targeted candidate shortlist.
