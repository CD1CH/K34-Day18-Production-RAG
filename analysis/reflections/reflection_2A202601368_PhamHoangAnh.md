# Individual Reflection — Lab 18: Production RAG Pipeline

**Học viên:** Phạm Hoàng Anh  
**Mã học viên / MSSV:** 2A202601368  
**Lớp:** AICB-K34  
**Module phụ trách:** Toàn bộ Pipeline (M1 → M5)

---

## Phần 1: Mapping bài giảng (Lecture → Code)

| Khái niệm bài giảng | Module | Hàm cụ thể | Quan sát & Nhận xét thực tế |
|---|---|---|---|
| **Semantic Chunking** | M1 | `chunk_semantic()` | Dùng cosine similarity giữa các câu liền kề (`all-MiniLM-L6-v2`, threshold=0.85). Giữ nguyên mạch ý, không bị ngắt quãng giữa các điều khoản chính sách. |
| **Hierarchical Chunking** | M1 | `chunk_hierarchical()` | Tạo cặp Parent (2048 chars) và Child (256 chars). Retrieve child cho độ chính xác cao nhưng trả về context parent đầy đủ cho LLM. |
| **Structure-Aware Chunking** | M1 | `chunk_structure_aware()` | Parse theo Markdown headers (`#`, `##`, `###`). Giữ nguyên vẹn bảng biểu quy định và danh sách con. |
| **Vietnamese Word Segmentation** | M2 | `segment_vietnamese()` | Tách từ bằng `underthesea`, thay dấu `_` bằng khoảng trắng để BM25Okapi khớp chuẩn xác giữa query và document. |
| **BM25 + Dense Fusion (Hybrid)** | M2 | `reciprocal_rank_fusion()` | Dùng công thức RRF `score(d) = Σ 1/(60 + rank)` để dung hợp kết quả keyword search và vector search mà không cần chuẩn hóa scale điểm. |
| **Cross-Encoder Reranking** | M3 | `CrossEncoderReranker.rerank()` | Dùng `BAAI/bge-reranker-v2-m3` chấm điểm tương quan sâu theo cặp `(query, passage)`, giảm từ top-20 xuống top-3 chunks chuẩn nhất. |
| **RAGAS 4 Metrics Evaluation** | M4 | `evaluate_ragas()` | Đánh giá toàn diện 4 trụ cột: Faithfulness (0.8026), Answer Relevancy (0.8677), Context Precision (0.9333), Context Recall (0.8250). |
| **Diagnostic Failure Tree** | M4 | `failure_analysis()` | Tự động phân loại nguyên nhân lỗi (Hallucination, Missing Chunks, Irrelevant Chunks, Ambiguous Prompt) và gợi ý cách fix. |
| **Contextual Prepend & Enrichment** | M5 | `_enrich_single_call()` / `contextual_prepend()` | Gắn bối cảnh tài liệu vào đầu mỗi chunk kết hợp tạo câu hỏi giả định (HyQA) và metadata tự động giúp Answer Relevancy tăng vượt bậc (+19.5%). |

---

## Phần 2: Khó khăn & Cách giải quyết

1. **Lỗi Tokenizer Tiếng Việt trong BM25:**
   - *Lỗi:* `underthesea.word_tokenize` sinh từ ghép dạng `nghỉ_phép`. Khi query tìm kiếm gõ `nghỉ phép`, BM25 split khoảng trắng thành 2 token riêng biệt nên không khớp được từ khóa `nghỉ_phép`.
   - *Giải pháp:* Thay thế dấu gạch dưới `.replace("_", " ")` trước khi đưa vào index và query BM25.
2. **Lỗi Xung đột Phiên bản Tài liệu (Document Versioning):**
   - *Lỗi:* Trong corpus có cả `mat_khau_v1.md` (đổi sau 90 ngày) và `mat_khau_v2.md` (đổi sau 120 ngày). Khi truy vấn, hệ thống retrieve cả 2 tài liệu khiến LLM bị phân vân hoặc trả lời cả 2 mốc thời gian (Faithfulness giảm).
   - *Giải pháp:* Áp dụng metadata filter hoặc bổ sung thông tin phiên bản hiệu lực vào Contextual Prepend trong Module 5.
3. **Lỗi Môi trường & Dependency:**
   - *Lỗi:* Thiếu thư viện `scikit-learn` trong `.venv` và cổng Qdrant chưa kết nối.
   - *Giải pháp:* Cài lại bản `scikit-learn==1.5.2` ổn định và kích hoạt Docker Qdrant qua `docker compose up -d`.

---

## Phần 3: Action Plan cho Project

### Project: Trợ lý AI Tra cứu Chính sách & Quy trình Doanh nghiệp (Enterprise Policy Assistant)

### 1. Hiện trạng & Thách thức
- RAG pipeline cơ bản thường bị cắt vụn câu chữ, không nhận diện được bảng biểu phân cấp thẩm quyền tài chính và dễ nhầm lẫn tài liệu chính sách cũ đã hết hiệu lực.

### 2. Kế hoạch áp dụng các kỹ thuật từ Lab 18
1. **Chunking Strategy:** Kết hợp **Structure-Aware** cho tài liệu văn bản hành chính (theo điều khoản/chương mục) và **Hierarchical Chunking** (Parent 2048 / Child 256) cho cẩm nang dài.
2. **Search Strategy:** Bắt buộc dùng **Hybrid Search** (BM25 tách từ tiếng Việt + Dense Search `BAAI/bge-m3` trên Qdrant) ghép nối qua **RRF** để vừa bắt đúng từ khóa chuyên ngành vừa hiểu ngữ nghĩa.
3. **Reranking:** Tích hợp `bge-reranker-v2-m3` cho các truy vấn quan trọng, hoặc dùng `flashrank` nếu yêu cầu latency dưới 10ms.
4. **Enrichment:** Áp dụng **Contextual Prepend** (Anthropic style) ghi rõ số hiệu văn bản và ngày ban hành để triệt tiêu lỗi phiên bản tài liệu.
5. **Evaluation:** Thiết lập CI/CD định kỳ chạy **RAGAS** với tập 50+ test cases thực tế để giám sát độ trôi (drift) chất lượng trả lời.

### 3. Timeline triển khai
- **Tuần 1:** Chuẩn hóa dữ liệu corpus, xây dựng module Structure Chunking và đánh chỉ mục Qdrant Hybrid.
- **Tuần 2:** Tích hợp M5 Enrichment và Cross-Encoder Reranking.
- **Tuần 3:** Xây dựng bộ test set benchmark RAGAS, tuning ngưỡng lọc điểm và hoàn thiện giao diện chat.
