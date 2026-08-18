# Individual Reflection — Lab 18

**Tên:** Nguyen Van Thanh  
**Module phụ trách:** All 5 modules (M1-M5)  
**Ngày hoàn thành:** 2026-08-18  
**Pipeline run time:** 2265s end-to-end (RAGAS + 5 modules)

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

## Project: HR Policy RAG Chatbot (internal tool cho ~200 nhân viên)

### Hiện tại
- RAG pipeline hiện tại: Chưa có — đang xây mới
- Known issues: Dữ liệu HR policy nằm rải rác trong nhiều file PDF/markdown, có nhiều version (v2023 vs v2024); nhân viên hay hỏi trùng lặp về nghỉ phép, bảo hiểm, MFA

### Plan áp dụng
1. [x] **Chunking strategy:** Structure-Aware (M1) — parse markdown headers để giữ theo section, kèm metadata `policy_version` để phân biệt v2023 vs v2024
2. [x] **Search:** Hybrid (BM25 + Dense + RRF) — bge-m3 dense + underthesea BM25, đặc biệt quan trọng với tiếng Việt
3. [x] **Reranking:** Cross-encoder (BAAI/bge-reranker-v2-m3) — tăng precision, kết quả thực tế precision tăng từ 0.94 → 0.946
4. [x] **Evaluation:** RAGAS 4 metrics — đã chạy được, faithfulness 0.67, context_precision 0.95, context_recall 0.78
5. [x] **Enrichment:** Combined single-call (M5) — 107 chunks được enrich, giảm noise

### Pipeline run thực tế (Lab 18)
- **Baseline:** faithfulness 0.68, context_precision 0.94, context_recall 0.86
- **Production:** faithfulness 0.67, context_precision 0.95, context_recall 0.78
- **Δ:** +0.005 precision, -0.083 recall (rerank top-3 giảm recall — cần tăng top_k)

### Timeline
- **Tuần 1:** Setup infrastructure (Docker Qdrant, API keys, ingest 30+ HR policies)
- **Tuần 2:** Apply M1 structure-aware chunking với metadata version
- **Tuần 3:** M2 hybrid search + M3 rerank (tăng top_k=5 để cải thiện recall)
- **Tuần 4:** M5 enrichment, M4 RAGAS evaluation trên 50 test cases
- **Tuần 5:** Deploy lên Slack bot cho nhân viên công ty

### Bài học chính từ Lab 18
1. **Reranking tăng precision nhưng giảm recall** — production cần balance bằng `top_k` lớn hơn
2. **Metadata version là chìa khóa** cho policy documents — bottom-5 failures đều liên quan policy cũ vs mới
3. **Vietnamese BM25 cần underthesea** segmentation, không dùng whitespace tokenize
4. **RAGAS API không stable** — versions ragas 0.1.x → 0.4.x có breaking changes; cần `result.scores` thay vì `to_pandas()`
5. **Combined enrichment single-call** tiết kiệm 4× API cost so với 4 calls riêng
