# Failure Analysis — Lab 18

**Người thực hiện:** Nguyen Van Thanh  
**Ngày chạy:** 2026-08-18

## RAGAS Scores

| Metric | Naive Baseline | Production | Δ |
|--------|---------------|------------|---|
| Faithfulness | 0.6833 | 0.6667 | -0.0167 |
| Answer Relevancy | 0.0000* | 0.0000* | +0.0000 |
| Context Precision | 0.9417 | **0.9458** | +0.0042 ✓ |
| Context Recall | **0.8583** | 0.7750 | -0.0833 |

*\*answer_relevancy = 0 do OpenAIEmbeddings API mismatch (ragas 0.4.x dùng API mới `embed_documents` thay vì `embed_query` mà ragas vẫn gọi tên cũ). Metric này không phản ánh chất lượng pipeline.*

## Nhận xét tổng quan

- **Context Precision** tăng nhẹ (+0.0042) → Reranking (M3) + Enrichment (M5) giúp loại bớt chunks không liên quan.
- **Context Recall** giảm (-0.0833) → Do top-3 sau rerank bị giảm recall so với dense search trả về nhiều chunks hơn. Cần tăng `top_k` cho reranking.
- **Faithfulness** gần như không đổi → Pipeline end-to-end vẫn faithful với context.

## Bottom-5 Failures

### #1 — Lỗi Faithfulness (mật khẩu tối thiểu bao nhiêu ký tự?)
- **Question:** Mật khẩu phải có tối thiểu bao nhiêu ký tự?
- **Expected:** 12 ký tự (chính sách v2.0 hiện hành)
- **Got:** "Không tìm thấy."
- **Worst metric:** faithfulness (0.0)
- **Error Tree:**
  ```
  Output sai ("Không tìm thấy")
   → Context đúng? → KHÔNG — chunk bị bỏ qua
   → Query OK? → OK
   → Root cause: M1 chunking không bắt được đoạn văn bản liên quan, hoặc BM25 keyword "ký tự" không match BM25 Vietnamese segmentation
  ```
- **Diagnosis:** Missing relevant chunks (context_recall = 0.0)
- **Suggested fix:**
  - Tăng `top_k` cho BM25 + Dense
  - Hybrid search RRF với k=30 thay vì k=10
  - Thử semantic chunking (M1) để bắt đoạn về policy

### #2 — Lỗi Faithfulness (bao lâu đổi mật khẩu)
- **Question:** Bao lâu phải đổi mật khẩu một lần?
- **Expected:** 120 ngày (chính sách v2.0)
- **Got:** "Không tìm thấy."
- **Worst metric:** faithfulness (0.0)
- **Error Tree:**
  ```
  Output sai
   → Context đúng? → KHÔNG
   → Query OK? → OK
   → Root cause: Cùng vấn đề với #1 — văn bản về mật khẩu nằm trong chunks không được retrieve
  ```
- **Diagnosis:** Missing relevant chunks
- **Suggested fix:** Cùng với #1 — cải thiện BM25 hoặc thêm metadata filter

### #3 — Lỗi Context Recall (MFA)
- **Question:** Có cần kích hoạt xác thực đa yếu tố (MFA) không?
- **Expected:** Có, bắt buộc cho email/VPN/hệ thống nội bộ
- **Got:** Câu trả lời đúng nhưng context_recall = 0.5
- **Worst metric:** context_recall (0.5)
- **Error Tree:**
  ```
  Output OK nhưng context chưa đủ
   → Context đúng? → Một nửa, thiếu một số chunks
   → Query OK? → OK
   → Root cause: Reranker (M3) chỉ giữ top-3, bỏ qua chunks có thông tin chi tiết
  ```
- **Diagnosis:** Missing relevant chunks
- **Suggested fix:** Tăng top_k cho reranking (5 thay vì 3)

### #4 — Lỗi Context Recall (thâm niên phép năm)
- **Question:** Thâm niên bao nhiêu năm thì được cộng thêm ngày phép?
- **Expected:** 3 năm (chính sách v2024)
- **Got:** 5 năm (chính sách cũ v2023)
- **Worst metric:** context_recall (0.5)
- **Error Tree:**
  ```
  Output sai — trả về policy CŨ thay vì policy MỚI
   → Context đúng? → Retrieve cả 2 policies, nhưng top-1 là policy cũ
   → Query OK? → OK
   → Root cause: BM25 hoặc dense search rank policy cũ cao hơn policy mới
  ```
- **Diagnosis:** Missing relevant chunks / outdated policy conflict
- **Suggested fix:**
  - Thêm metadata `version=v2024` cho chunks mới
  - Filter theo metadata để ưu tiên policy hiện hành

### #5 — Lỗi Context Precision (thử việc nghỉ phép)
- **Question:** Nhân viên thử việc có được nghỉ phép năm không?
- **Expected:** KHÔNG
- **Got:** Đúng "KHÔNG" nhưng context_precision cao bất thường
- **Worst metric:** context_precision (≈1.0)
- **Error Tree:**
  ```
  Output OK
   → Context đúng? → Có nhiều chunks về "thử việc" nhưng có chunks không trực tiếp về "nghỉ phép"
   → Root cause: BM25 keyword "thử việc" match nhiều docs, một số không liên quan trực tiếp
  ```
- **Diagnosis:** Too many irrelevant chunks (top-3 chứa chunks noise)
- **Suggested fix:** Metadata filter theo topic hoặc add `parent_id` để gom theo section

## Case Study (presentation)

**Question:** "Nhân viên được nghỉ bao nhiêu ngày phép năm?"

**Error Tree walkthrough:**
1. Output đúng? → ✓ "15 ngày" (đúng theo policy v2024)
2. Context đúng? → ⚠ context_precision = 0.58 (có cả policy cũ 12 ngày)
3. Query rewrite OK? → ✓
4. Fix ở bước: **Chunking + Metadata** — đánh dấu version cho từng policy chunk

**Nếu có thêm 1 giờ:**
- Thêm metadata `policy_version` để filter chunks
- Tăng rerank top_k lên 5
- Test với semantic chunking thay vì paragraph để xem precision tăng không

## Implementation Summary

### Modules Implemented
- **M1 Chunking:** Semantic, Hierarchical, Structure-Aware (3 strategies)
- **M2 Search:** BM25 Vietnamese + Dense + RRF fusion
- **M3 Reranking:** Cross-encoder (BAAI/bge-reranker-v2-m3)
- **M4 Evaluation:** RAGAS 4 metrics + failure analysis
- **M5 Enrichment:** Combined single-call + 4 individual techniques

### Test Results
- **37/37 tests passing** (100%)
- M1: 13/13, M2: 5/5, M3: 5/5, M4: 4/4, M5: 10/10

### Pipeline Run
- **Status:** ✅ End-to-end success
- **Total time:** 2265s (~38 phút)
- **Documents indexed:** 26 (skip 2 PDF scan)
- **Chunks after M1:** 107
- **Chunks after M5:** 107 (with enrichment)
- **Test set:** 20 questions