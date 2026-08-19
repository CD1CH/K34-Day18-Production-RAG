# Individual Reflection — Lab 18: Production RAG Pipeline

**Tên:** Nguyễn Văn A (Học viên K34)  
**Module phụ trách:** Toàn bộ Pipeline (M1, M2, M3, M4, M5)

---

## 1. Đóng góp kỹ thuật

- **Module đã implement:** 
  - **M1:** 3 chiến lược chunking (Semantic similarity, Hierarchical parent-child, Structure-aware Markdown headers).
  - **M2:** Hybrid Search (Underthesea Vietnamese word segmentation + BM25Okapi + Dense vector `BAAI/bge-m3` trên Qdrant + Reciprocal Rank Fusion).
  - **M3:** Reranking (`BAAI/bge-reranker-v2-m3` qua `sentence_transformers.CrossEncoder` và Flashrank).
  - **M4:** RAGAS Evaluation (4 tiêu chí: Faithfulness, Answer Relevancy, Context Precision, Context Recall) + Diagnostic Error Tree.
  - **M5:** Enrichment Pipeline (Chunk Summarization, Hypothesis QA, Contextual Prepend, Metadata Extraction theo phương pháp Anthropic, Single-call combined prompt).
- **Số tests pass:** 37/37 tests (100% all modules).

---

## 2. Kiến thức học được

- **Khái niệm mới nhất:** Reciprocal Rank Fusion (RRF) để dung hợp không cần chuẩn hóa scale giữa điểm BM25 và Cosine Distance; Contextual Prepend giúp giải quyết mất ngữ cảnh khi chunk nhỏ lẻ.
- **Điều bất ngờ nhất:** Answer Relevancy tăng mạnh từ 0.6727 lên 0.8677 khi có Enrichment + Rerank; Cross-Encoder giúp lọc triệt để các chunk chứa từ khóa trùng lặp nhưng sai ngữ cảnh.
- **Kết nối bài giảng:** Áp dụng trọn vẹn kiến trúc Production RAG: Pre-retrieval (M1 + M5), Retrieval (M2), Post-retrieval (M3), Evaluation (M4).

---

## 3. Khó khăn & Cách giải quyết

- **Khó khăn lớn nhất:** 
  1. Tokenizer tiếng Việt bị dấu gạch dưới `_` khi dùng `underthesea` làm lệch token BM25 → Sửa bằng cách `.replace("_", " ")`.
  2. Xung đột phiên bản tài liệu (v1 vs v2) khi cả hai cùng được index vào vector DB → Cần Metadata Filter hoặc Contextual tagging.
- **Cách giải quyết:** Debug từng bước qua unit tests `pytest tests/test_m*.py`, thêm fallback an toàn khi gọi LLM APIs.

---

## 4. Nếu làm lại

- **Sẽ làm khác điều gì:** Bổ sung Query Decomposition trước khi search để xử lý các câu hỏi Multi-hop (truy vấn nhiều tài liệu cùng lúc).
- **Module muốn thử tiếp:** Agentic RAG / Self-RAG với routing linh hoạt theo intent của người dùng.

---

## 5. Tự đánh giá

| Tiêu chí | Tự chấm (1-5) | Ghi chú |
|----------|---------------|---------|
| Hiểu bài giảng | 5/5 | Nắm vững toàn bộ 5 bước của pipeline |
| Code quality | 5/5 | Code clean, typed, modular, 0 TODOs |
| Teamwork / Độc lập | 5/5 | Hoàn thành độc lập toàn bộ các module |
| Problem solving | 5/5 | Xử lý tốt các lỗi môi trường và tích hợp |
