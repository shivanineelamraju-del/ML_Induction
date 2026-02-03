# ML_Induction

# Analysis & Insights

## 1. Impact of Fine-Tuning on Retrieval Quality

Fine-tuning the pre-trained **all-MiniLM-L6-v2** sentence embedding model on MS MARCO query–passage relevance pairs led to consistent improvements across most ranking metrics compared to the pre-trained baseline.

Notably, fine-tuning improved:

* **Mean Reciprocal Rank (MRR)**, indicating that the first relevant document appears higher in the ranking.
* **NDCG@5 and NDCG@10**, showing better ordering of relevant documents among the top results.
* **Recall@10 and Recall@100**, suggesting improved coverage of relevant passages in the retrieved set.

These gains demonstrate that domain-specific supervision is critical for dense retrieval performance.

---

## 2. Why Fine-Tuning Improves Performance

The pre-trained MiniLM model is optimized for general sentence similarity and is not explicitly trained for information retrieval tasks.

Fine-tuning aligns the embedding space with the retrieval objective by:

* Pulling relevant query–document pairs closer together.
* Pushing irrelevant passages away using in-batch negatives via **MultipleNegativesRankingLoss**.
* Adapting representations to web search style queries found in MS MARCO.

As a result, the fine-tuned model better captures relevance signals required for ranking, rather than just semantic relatedness.

---

## 3. Failure Case Analysis

Despite overall improvements, several failure cases were observed:

### 3.1 Lexical Sensitivity

Dense retrieval sometimes struggles with queries requiring **exact lexical matches**, such as:

* Proper nouns
* Numerical values
* Rare entities or identifiers

Example:

> Query: *“population of city with zip code 94107”*

The dense model often retrieves passages about cities in general but misses documents containing the exact zip code, highlighting the lack of lexical precision.

---

### 3.2 Over-Generalization

For highly specific queries, the model may retrieve topically related but **non-answering** passages. This occurs because dense embeddings optimize semantic similarity, not factual correctness.

---

## 4. Limitations of the Current System

* No explicit **hard negative mining** beyond in-batch negatives.
* No **lexical retrieval component** (e.g., BM25), limiting performance on exact-match queries.
* Single-stage retrieval without re-ranking.
* Evaluation conducted on a **10k document subset**, which may not reflect large-scale deployment constraints.

---

## 5. Efficiency Considerations

* FAISS with `IndexFlatIP` performs exact nearest-neighbor search, which is efficient for small corpora but does not scale to millions of documents.
* Query latency is dominated by embedding computation rather than ANN search.
* Memory usage grows linearly with the number of indexed embeddings.

For larger corpora, approximate indices (IVF, HNSW) would be required.

---

## 6. Future Improvements

Based on observed limitations, the following enhancements are planned:

1. **Hybrid Retrieval**

   * Combine dense retrieval with BM25 using Reciprocal Rank Fusion (RRF) to address lexical failures.

2. **Hard Negative Mining**

   * Use FAISS to mine challenging negatives during training.

3. **Two-Stage Re-ranking**

   * Apply a cross-encoder to re-rank top-K candidates for higher precision.

4. **Scalability Evaluation**

   * Measure latency percentiles (p50, p95, p99) and throughput.
   * Analyze index size and memory usage.

---

## 7. Key Takeaways

* Fine-tuning is essential for strong dense retrieval performance.
* Dense models excel at semantic matching but lack lexical precision.
* Evaluation should consider both effectiveness and efficiency.
* Hybrid and multi-stage systems are necessary for production-grade search engines.

---
