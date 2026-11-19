[← /qc-analyze](15-qc-analyze.md) · [Explain Home](README.md) · [Next: /qc-design-test →](17-qc-design-test.md)

# 16 · `/qc-plan` — Trạm 2: Kế hoạch test theo rủi ro

> **Một câu.** Từ phân tích yêu cầu, lập **TEST_PLAN** ưu tiên theo rủi ro và nêu câu hỏi cần dev làm rõ.

---

## Vấn đề giải quyết

Không phải mọi scenario rủi ro ngang nhau. Trạm này xếp ưu tiên test theo rủi ro (tránh dàn trải) và gom câu hỏi cho dev trước khi thiết kế test.

---

## Vị trí & tiền đề

- **Vị trí:** Phase QC (trạm 2), sau `/qc-analyze`.
- **Tiền đề:** REQUIREMENT_ANALYSIS.md + DOC_GAP.md. Blocker 🔴 **không chặn** lập plan — nó chỉ làm UC đó thành `Blocked` ở §5 Entry criteria.
- **Phạm vi:** `uc_list` phải khớp bảng *Phạm vi phân tích* trong `DOC_GAP.md`; lệch (BDD đổi trạng thái sau trạm 1) → nêu ra, khuyên chạy lại `/qc-analyze`.

---

## Input / Output

**Input:** `{TICKET-ID} [platform]` (UC-ID / file feature cũng quy về TICKET-ID) + analysis từ trạm 1 + `.feature` của đúng nền + skill `qa-planner` (`test-plan.md` + `risk-model.md`).

**Output:** `{qc_dir}/{TICKET-ID}-{prd-slug}/{platform}/TEST_PLAN.md` — **một plan cho cả PRD** (rủi ro + câu hỏi cho dev), `Ready`/`Blocked` chấm theo từng UC.

---

## Các bước xử lý (chi tiết)

1. **Role qa-planner** — nạp `{qc_skills_dir}/qa-planner/`.
2. Đọc analysis + gap → **đánh giá rủi ro** từng vùng theo `risk-model.md` (7 nguồn rủi ro · khả năng × thiệt hại → **P0–P3**, dùng mức đó chia độ sâu test).
3. Nêu **câu hỏi cho dev** (`questions-for-dev`, suy ra từ gap open/blocker).
4. **Self-Review** *(mới)* — nạp `skills/qc/_shared/self-review-principles.md`, soát một lượt trước khi in report.
5. **Output** TEST_PLAN.md — bắt buộc: bảng `§3 Test items` có cột `UC`; `§2 Phạm vi` liệt kê UC `⏸ Chưa xét` ở *Out of scope*; `§5 Entry criteria` ghi `Ready`/`Blocked` **cho từng UC**. Append một dòng `"phase":"qc-plan"` vào `quality-metrics.jsonl`.
> **Self-Review ≠ Guard.** Guard là phép **đếm cơ học**, có hệ quả bắt buộc khi lệch. Self-review là lượt đọc lại **rộng hơn nhưng mềm hơn**, soát ba nhóm lỗi mà phép đếm không bắt được (bịa dữ kiện · lẫn suy đoán với sự thật · bỏ dở giữa chừng). Một bộ nguyên tắc tự soát **không bao giờ** được dùng làm lý do gỡ một Guard — chính file `self-review-principles.md` ghi rõ ranh giới đó ngay ở đầu.


---

## Checkpoint & Gate

- **CHECKPOINT: chặn thường** — `--yes` bỏ qua được. Ghi đè `TEST_PLAN.md` là **an toàn**: không ô nào người điền tay (câu trả lời quay về qua `DOC_GAP.md`), mọi cột do lệnh tự tính → chạy lại = lập lại. Có ai bắt đầu điền tay vào file này thì phải nâng lên chặn CỨNG.
- **`Blocked` là mức ưu tiên, không phải lệnh cấm** — `/qc-design-test` vẫn chạy trên UC `Blocked`; cái bị chặn là *chạy* test, không phải *viết* test.

---

## Cơ chế đặc biệt

- **Risk-based** — kế hoạch bám rủi ro, không phủ đều.
- **Câu hỏi cho dev** — kênh QC ↔ Dev có hồ sơ trước khi viết test.
- **Self-Review dùng chung** — mọi trạm QC nạp **cùng một** file nguyên tắc, không mỗi trạm một bản. Mỗi trạm một bản sao thì sáu tháng sau sẽ là từng ấy luật khác nhau, lệch nhau mà không ai được cảnh báo (lỗi ngầm).

---

## 👓 Góc nhìn tối ưu

- **Trạm mỏng nhất** (~thin orchestrator + skill). Giá trị phụ thuộc chất lượng skill qa-planner.
- **Câu hỏi cho dev** không có cơ chế theo dõi trả lời — dễ rơi. Cân nhắc gắn vào DOC_GAP/feedback.

---

## Kết nối

**Trước:** [`/qc-analyze`](15-qc-analyze.md) · **Sau:** [`/qc-design-test`](17-qc-design-test.md).
