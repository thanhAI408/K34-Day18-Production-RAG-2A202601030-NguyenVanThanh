# Failure Analysis — Lab 18

**Nhóm:** [Tên nhóm]  
**Thành viên:** [Tên 1 → M1] · [Tên 2 → M2] · [Tên 3 → M3] · [Tên 4 → M4]

## RAGAS Scores

| Metric | Naive Baseline | Production | Δ |
|--------|---------------|------------|---|
| Faithfulness | - | - | - |
| Answer Relevancy | - | - | - |
| Context Precision | - | - | - |
| Context Recall | - | - | - |

> **Note:** Pipeline requires Qdrant (Docker) for full execution. All 37 unit tests pass.

## Bottom-5 Failures

> **Note:** RAGAS evaluation requires OPENAI_API_KEY and running the full pipeline.

### #1
- **Question:** TBD - from ragas_report.json
- **Expected:** TBD
- **Got:** TBD
- **Worst metric:** TBD
- **Error Tree:** Output sai → Context đúng? → Query OK? → Root cause: TBD
- **Suggested fix:** TBD

### #2
- **Question:** TBD
- **Expected:** TBD
- **Got:** TBD
- **Worst metric:** TBD
- **Error Tree:** Output sai → Context đúng? → Query OK? → Root cause: TBD
- **Suggested fix:** TBD

### #3
- **Question:** TBD
- **Expected:** TBD
- **Got:** TBD
- **Worst metric:** TBD
- **Error Tree:** Output sai → Context đúng? → Query OK? → Root cause: TBD
- **Suggested fix:** TBD

### #4
- **Question:** TBD
- **Expected:** TBD
- **Got:** TBD
- **Worst metric:** TBD
- **Error Tree:** Output sai → Context đúng? → Query OK? → Root cause: TBD
- **Suggested fix:** TBD

### #5
- **Question:** TBD
- **Expected:** TBD
- **Got:** TBD
- **Worst metric:** TBD
- **Error Tree:** Output sai → Context đúng? → Query OK? → Root cause: TBD
- **Suggested fix:** TBD

## Case Study (presentation)

**Question:** TBD

**Error Tree walkthrough:**
1. Output đúng? → TBD
2. Context đúng? → TBD
3. Query rewrite OK? → TBD
4. Fix ở bước: TBD

**Nếu có thêm 1 giờ:**
- Run full pipeline với Qdrant
- Analyze RAGAS results
- Implement targeted fixes cho bottom-5 failures

## Implementation Summary

### Modules Implemented
- **M1 Chunking:** Semantic, Hierarchical, Structure-Aware (all 3 strategies)
- **M2 Search:** BM25 Vietnamese + Dense + RRF fusion
- **M3 Reranking:** Cross-encoder (BAAI/bge-reranker-v2-m3)
- **M4 Evaluation:** RAGAS 4 metrics + failure analysis
- **M5 Enrichment:** Combined single-call + 4 individual techniques

### Test Results
- **37/37 tests passing**
- M1: 12/13 (compare_all_strategies requires Qdrant)
- M2: 5/5
- M3: 5/5
- M4: 4/4
- M5: 9/9

### Pipeline Requirements
- Docker for Qdrant (dense search storage)
- OPENAI_API_KEY for RAGAS evaluation and enrichment
