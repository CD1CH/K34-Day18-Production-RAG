# Group Report — Lab 18: Production RAG

**Nhóm:** K34 Production RAG  
**Ngày:** 19/08/2026

## Thành viên & Phân công

| Tên | MSSV | Module | Hoàn thành | Tests pass |
|-----|------|--------|-----------|-----------|
| Phạm Hoàng Anh | 2A202601368 | M1: Chunking | ✅ | 13/13 |
| Phạm Hoàng Anh | 2A202601368 | M2: Hybrid Search | ✅ | 5/5 |
| Phạm Hoàng Anh | 2A202601368 | M3: Reranking | ✅ | 5/5 |
| Phạm Hoàng Anh | 2A202601368 | M4: Evaluation | ✅ | 4/4 |
| Phạm Hoàng Anh | 2A202601368 | M5: Enrichment | ✅ | 10/10 |

## Kết quả RAGAS

| Metric | Naive | Production | Δ |
|--------|-------|-----------|---|
| Faithfulness | 0.8158 | 0.8026 | -0.0132 |
| Answer Relevancy | 0.6727 | 0.8677 | **+0.1950** |
| Context Precision | 0.9333 | 0.9333 | +0.0000 |
| Context Recall | 0.9250 | 0.8250 | -0.1000 |

## Key Findings

1. **Biggest improvement:** Answer Relevancy tăng vượt bậc từ **0.6727 lên 0.8677 (+0.1950)** nhờ HyQA + Contextual Prepend trong Module 5 và Reranking trong Module 3.
2. **Biggest challenge:** Xử lý các câu hỏi Multi-hop (cần 2 tài liệu độc lập) và câu hỏi dính Document Version conflict (chính sách cũ vs mới).
3. **Surprise finding:** Hybrid Search kết hợp BM25 (word segmentation tiếng Việt) và Dense Search qua RRF đem lại Context Precision rất cao (0.9333).

## Presentation Notes (5 phút)

1. **RAGAS scores (naive vs production):** Relevancy tăng mạnh (+20%), Precision đạt 93.3%.
2. **Biggest win — module nào, tại sao:** M5 Enrichment + M3 Rerank giúp câu trả lời tập trung trực diện vào ý hỏi của người dùng.
3. **Case study — 1 failure, Error Tree walkthrough:** Xung đột chính sách đổi mật khẩu 90 ngày (v1) vs 120 ngày (v2) → Cần Metadata Filter.
4. **Next optimization nếu có thêm 1 giờ:** Thêm Query Decomposition và Document Versioning Filter.

