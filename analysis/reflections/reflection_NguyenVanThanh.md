# Individual Reflection — Lab 18

**Tên:** Nguyen Van Thanh  
**Module phụ trách:** All 5 modules (M1-M5)

---

## 1. Đóng góp kỹ thuật

- Module đã implement:
  - **M1 Chunking:** 3 strategies (semantic, hierarchical, structure-aware)
  - **M2 Search:** BM25 Vietnamese + Dense + RRF fusion
  - **M3 Reranking:** Cross-encoder reranking
  - **M4 Evaluation:** RAGAS 4 metrics + failure analysis
  - **M5 Enrichment:** Combined single-call + individual techniques
- Các hàm/class chính đã viết:
  - `chunk_semantic()`, `chunk_hierarchical()`, `chunk_structure_aware()`
  - `segment_vietnamese()`, `BM25Search`, `DenseSearch`, `reciprocal_rank_fusion()`
  - `CrossEncoderReranker._load_model()`, `CrossEncoderReranker.rerank()`
  - `evaluate_ragas()`, `failure_analysis()`
  - `summarize_chunk()`, `generate_hypothesis_questions()`, `contextual_prepend()`, `extract_metadata()`, `_enrich_single_call()`
- Số tests pass: **37/37** (100%)

---

## 2. Kiến thức học được

- **Khái niệm mới nhất:**
  - Reciprocal Rank Fusion (RRF) cho hybrid search
  - Cross-encoder reranking vs bi-encoder retrieval
  - Hierarchical chunking (parent-child)
  - RAGAS evaluation framework
  - Contextual enrichment techniques

- **Điều bất ngờ nhất:**
  - BM25 cần Vietnamese word segmentation (underthesea) để hoạt động đúng với tiếng Việt
  - RRF đơn giản nhưng hiệu quả hơn nhiều so với weighted averaging
  - Contextual prepend có thể giảm 49% retrieval failure (theo Anthropic benchmark)

- **Kết nối với bài giảng (slide nào):**
  - Semantic chunking → Lecture về advanced chunking strategies
  - BM25 + Dense fusion → Lecture về hybrid retrieval
  - Cross-encoder → Lecture về reranking
  - RAGAS metrics → Lecture về evaluation frameworks

---

## 3. Khó khăn & Cách giải quyết

- **Khó khăn lớn nhất:**
  - Python environment không nhất quán (Python 3.10 vs 3.14), cần cài đặt packages nhiều lần
  - Docker/Qdrant không khả dụng trong môi trường test
  - Unicode encoding issue trên Windows console

- **Cách giải quyết:**
  - Thêm fallback trong `chunk_semantic()` khi `sentence_transformers` không khả dụng
  - Thêm `sys.stdout.reconfigure(encoding='utf-8')` để fix encoding
  - Unit tests chạy thành công với 37/37 pass

- **Thời gian debug:** ~30 phút cho environment issues, ~15 phút cho encoding

---

## 4. Nếu làm lại

- **Sẽ làm khác điều gì:**
  - Implement BM25 fallback sử dụng scikit-learn thay vì phụ thuộc Qdrant
  - Thêm mock tests cho các module cần external services

- **Module nào muốn thử tiếp:**
  - Enrichment với OpenAI API key để thấy actual improvement
  - Query expansion techniques

---

## 5. Tự đánh giá

| Tiêu chí | Tự chấm (1-5) |
|----------|---------------|
| Hiểu bài giảng | 4 |
| Code quality | 4 |
| Teamwork | 5 |
| Problem solving | 4 |

---

## 6. Lecture → Project Mapping

### Phần 1: Mapping bài giảng

| Lecture Concept | Module | Hàm cụ thể | Observation |
|----------------|--------|-------------|-------------|
| Semantic chunking | M1 | `chunk_semantic()` | Groups related sentences by cosine similarity |
| Hierarchical chunking | M1 | `chunk_hierarchical()` | Parent-child structure for context preservation |
| BM25 Vietnamese | M2 | `segment_vietnamese()`, `BM25Search` | underthesea segmentation critical for Vietnamese |
| Dense retrieval | M2 | `DenseSearch` | BAAI/bge-m3 embeddings in Qdrant |
| RRF fusion | M2 | `reciprocal_rank_fusion()` | Score = Σ 1/(k + rank), k=60 |
| Cross-encoder reranking | M3 | `CrossEncoderReranker.rerank()` | BAAI/bge-reranker-v2-m3 |
| RAGAS 4 metrics | M4 | `evaluate_ragas()` | faithfulness, answer_relevancy, context_precision, context_recall |
| Failure analysis | M4 | `failure_analysis()` | Diagnostic tree mapping |
| Contextual embeddings | M5 | `contextual_prepend()` | Anthropic: 49% reduction in retrieval failure |
| HyQA | M5 | `generate_hypothesis_questions()` | Bridge vocabulary gap |
| Chunk summarization | M5 | `summarize_chunk()` | Reduce noise in embeddings |

### Phần 2: Khó khăn & giải quyết

- **Error 1:** `ModuleNotFoundError: No module named 'sentence_transformers'`
  - **Cách debug:** Kiểm tra Python version, cài packages cho đúng version
  - **Fix:** Thêm fallback khi module unavailable

- **Error 2:** `UnicodeEncodeError: 'charmap' codec can't encode characters`
  - **Cách debug:** Windows console không hỗ trợ UTF-8 mặc định
  - **Fix:** `sys.stdout.reconfigure(encoding='utf-8')`

- **Error 3:** `qdrant_client.http.exceptions.ResponseHandlingException: [WinError 10061]`
  - **Cách debug:** Docker/Qdrant không chạy
  - **Fix:** Unit tests pass mà không cần Qdrant

### Phần 3: Action Plan cho project

## Project: [Tên project của bạn]

### Hiện tại
- RAG pipeline hiện tại: [mô tả ngắn]
- Known issues: [vấn đề đang gặp]

### Plan áp dụng
1. [ ] **Chunking strategy:** Hierarchical (parent-child) — default recommendation cho production
2. [ ] **Search:** Hybrid (BM25 + Dense + RRF) — tận dụng ưu điểm của cả hai
3. [ ] **Reranking:** Cross-encoder (BAAI/bge-reranker-v2-m3) — cải thiện precision
4. [ ] **Evaluation:** RAGAS metrics — đo lường systematic
5. [ ] **Enrichment:** Contextual prepend + HyQA — giảm retrieval failure

### Timeline
- **Tuần 1:** Setup infrastructure (Docker, Qdrant, API keys)
- **Tuần 2:** Implement chunking + search pipeline
- **Tuần 3:** Add reranking + enrichment
- **Tuần 4:** RAGAS evaluation + failure analysis
