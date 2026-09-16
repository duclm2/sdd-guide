[← /qc-analyze](15-qc-analyze.md) · [Explain Home](README.md) · [Next: /qc-design-test →](17-qc-design-test.md)

# 16 · `/qc-plan` — Trạm 2: Kế hoạch test theo rủi ro

> **Một câu.** Từ phân tích yêu cầu, lập **TEST_PLAN** ưu tiên theo rủi ro và nêu câu hỏi cần dev làm rõ.

---

## Vấn đề giải quyết

Không phải mọi scenario rủi ro ngang nhau. Trạm này xếp ưu tiên test theo rủi ro (tránh dàn trải) và gom câu hỏi cho dev trước khi thiết kế test.

---

## Vị trí & tiền đề

- **Vị trí:** Phase QC (trạm 2), sau `/qc-analyze`.
- **Tiền đề:** REQUIREMENT_ANALYSIS.md + DOC_GAP.md; nên xử blocker 🔴 trước.

---

## Input / Output

**Input:** analysis từ trạm 1 + skill `qa-planner`.

**Output:** `{qc_dir}/{TICKET-ID}/{platform}/TEST_PLAN.md` — **một plan cho cả PRD** (rủi ro + câu hỏi cho dev), `Ready`/`Blocked` chấm theo từng UC.

---

## Các bước xử lý (chi tiết)

1. **Role qa-planner** — nạp `{qc_skills_dir}/qa-planner/`.
2. Đọc analysis + gap → **đánh giá rủi ro** từng vùng.
3. Nêu **câu hỏi cho dev** (điểm chưa rõ để test đúng).
4. **Self-Review** *(mới)* — nạp `skills/qc/_shared/self-review-principles.md`, soát một lượt trước khi in report.
5. **Output** TEST_PLAN.md.
> **Self-Review ≠ Guard.** Guard là phép **đếm cơ học**, có hệ quả bắt buộc khi lệch. Self-review là lượt đọc lại **rộng hơn nhưng mềm hơn**, soát ba nhóm lỗi mà phép đếm không bắt được (bịa dữ kiện · lẫn suy đoán với sự thật · bỏ dở giữa chừng). Một bộ nguyên tắc tự soát **không bao giờ** được dùng làm lý do gỡ một Guard — chính file `self-review-principles.md` ghi rõ ranh giới đó ngay ở đầu.


---

## Checkpoint & Gate

- Không gate chặn.

---

## Cơ chế đặc biệt

- **Risk-based** — kế hoạch bám rủi ro, không phủ đều.
- **Câu hỏi cho dev** — kênh QC ↔ Dev có hồ sơ trước khi viết test.
- **Self-Review dùng chung** — sáu trạm QC nạp **cùng một** file nguyên tắc, không mỗi trạm một bản. Sáu bản sao thì sáu tháng sau sẽ là sáu luật khác nhau, **im lặng**.

---

## 👓 Góc nhìn tối ưu

- **Trạm mỏng nhất** (~thin orchestrator + skill). Giá trị phụ thuộc chất lượng skill qa-planner.
- **Câu hỏi cho dev** không có cơ chế theo dõi trả lời — dễ rơi. Cân nhắc gắn vào DOC_GAP/feedback.

---

## Kết nối

**Trước:** [`/qc-analyze`](15-qc-analyze.md) · **Sau:** [`/qc-design-test`](17-qc-design-test.md).
