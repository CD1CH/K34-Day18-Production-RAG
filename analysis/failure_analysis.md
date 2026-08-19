# Failure Analysis — Lab 18: Production RAG

**Nhóm:** K34 Production RAG  
**Thành viên:** Phạm Hoàng Anh (MSSV: 2A202601368) — Cá nhân thực hiện M1 → M5

---

## RAGAS Scores

| Metric | Naive Baseline | Production | Δ |
|--------|---------------|------------|---|
| Faithfulness | 0.8158 | 0.8026 | -0.0132 |
| Answer Relevancy | 0.6727 | 0.8677 | +0.1950 |
| Context Precision | 0.9333 | 0.9333 | +0.0000 |
| Context Recall | 0.9250 | 0.8250 | -0.1000 |

---

## Bottom-5 Failures

### #1
- **Question:** "Nhân viên tạm ứng 15 triệu, sau 20 ngày mới thanh toán. Bị phạt bao nhiêu?"
- **Expected:** Phạt lãi suất chậm thanh toán hoặc trừ lương theo quy định tạm ứng.
- **Got:** LLM hallucinate số tiền phạt không chính xác hoặc suy diễn ngoài văn bản.
- **Worst metric:** Faithfulness (0.2857)
- **Error Tree:** Output sai → Context đúng? (Có context tạm ứng nhưng thiếu chi tiết phạt ngày cụ thể) → LLM suy diễn thêm.
- **Root cause:** Context không có điều khoản phạt cụ thể nhưng LLM vẫn cố trả lời thay vì nói "Không tìm thấy".
- **Suggested fix:** Thắt chặt system prompt ("Chỉ trả lời nếu context có nêu rõ, không suy diễn").

### #2
- **Question:** "Muốn mua thiết bị trị giá 55 triệu cần ai phê duyệt?"
- **Expected:** Cấp phê duyệt theo hạn mức mua sắm tài sản (Tổng giám đốc / Giám đốc khối).
- **Got:** Trả lời nhầm cấp phê duyệt của hạn mức dưới 50 triệu.
- **Worst metric:** Faithfulness (0.0000)
- **Error Tree:** Output sai → Context đúng? (Có context bảng hạn mức nhưng LLM đọc nhầm mốc 50tr vs 55tr).
- **Root cause:** LLM nhầm lẫn boundary giữa các mốc phân cấp thẩm quyền tài chính.
- **Suggested fix:** Structure-aware chunking giữ nguyên bảng markdown và tăng nhiệt độ reranking cho mốc số liệu.

### #3
- **Question:** "Nếu cần mua một chiếc laptop 30 triệu cho nhân viên mới, ai phê duyệt và cần gì từ phòng CNTT?"
- **Expected:** Trưởng bộ phận phê duyệt + Phiếu yêu cầu kỹ thuật/cấu hình từ IT.
- **Got:** Trả lời đúng một phần thẩm quyền nhưng thiếu yêu cầu từ IT.
- **Worst metric:** Faithfulness (0.5000)
- **Error Tree:** Output thiếu 1 vế → Context đúng? (Context gồm 2 chunk khác nhau nhưng rerank ưu tiên 1 chunk).
- **Root cause:** Câu hỏi Multi-hop ghép 2 ý (phê duyệt chi tiêu + thủ tục IT).
- **Suggested fix:** Query decomposition (tách câu hỏi thành 2 sub-queries) để retrieval đầy đủ cả 2 phần.

### #4
- **Question:** "Một nhân viên Senior có 9 năm thâm niên được nghỉ bao nhiêu ngày phép năm và lương trong khoảng nào?"
- **Expected:** 16 ngày phép (15 ngày cơ bản + 1 ngày thâm niên) và mức lương Senior theo bảng lương.
- **Got:** Thiếu context bảng lương (chỉ tìm thấy context nghỉ phép).
- **Worst metric:** Context Recall (0.0000)
- **Error Tree:** Output thiếu vế lương → Context thiếu chunk bảng lương → Top-k rerank bị chiếm hết bởi chunk nghỉ phép.
- **Root cause:** Truy vấn Multi-hop cần 2 tài liệu độc lập (`nghi_phep` và `thang_bang_luong`).
- **Suggested fix:** Multi-query expansion hoặc tăng `RERANK_TOP_K` từ 3 lên 5.

### #5
- **Question:** "Bao lâu phải đổi mật khẩu một lần?"
- **Expected:** 120 ngày (theo chính sách v2 hiện hành) kèm MFA.
- **Got:** Nhắc đến cả 90 ngày (v1 cũ) và 120 ngày (v2 mới).
- **Worst metric:** Faithfulness (0.6000)
- **Error Tree:** Output phân vân 2 phiên bản → Context chứa cả tài liệu cũ và mới.
- **Root cause:** Xung đột phiên bản tài liệu (Document Version conflict: `mat_khau_v1.md` vs `mat_khau_v2.md`).
- **Suggested fix:** Áp dụng Metadata filtering (ưu tiên document có status `effective`/năm mới nhất) hoặc Contextual Enrichment ghi rõ phiên bản hiện hành.

---

## Case Study (cho presentation)

**Question chọn phân tích:** #"Bao lâu phải đổi mật khẩu một lần?"

**Error Tree walkthrough:**
1. Output đúng? → Không hoàn toàn (bị nhiễu bởi văn bản v1 cũ).
2. Context đúng? → Context thừa chunk phiên bản cũ (`mat_khau_v1.md`).
3. Query rewrite OK? → Chưa có metadata filter loại bỏ tài liệu superseded.
4. Fix ở bước: **M5 Enrichment / Metadata Filter** (Gắn tag `status: active/superseded` vào chunk metadata).

**Nếu có thêm 1 giờ, sẽ optimize:**
- Thêm **Document Versioning Filter** trong M2/M3 để tự động loại bỏ tài liệu cũ khi đã có tài liệu mới thay thế.
- Tích hợp **Query Decomposition** cho các câu hỏi Multi-hop (như câu #3, #4).
