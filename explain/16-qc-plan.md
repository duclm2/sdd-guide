[← /qc-analyze](15-qc-analyze.md) · [Explain Home](README.md) · [Next: /qc-design-test →](17-qc-design-test.md)

# 16 · `/qc-plan` — Trạm 2: Kế hoạch test theo rủi ro

> **Một câu.** Từ phân tích yêu cầu, lập **TEST_PLAN** ưu tiên theo rủi ro và nêu câu hỏi cần dev làm rõ.

---

## Vấn đề giải quyết

Không phải mọi scenario rủi ro ngang nhau. Trạm này xếp ưu tiên test theo rủi ro (tránh dàn trải) và gom câu hỏi cho dev trước khi thiết kế test.

---

## Vị trí & tiền đề

- **Vị trí:** Phase QC (trạm 2), sau `/qc-analyze`.
- **Tiền đề:** REQUIREMENT_ANALYSIS.md + DOC_GAPS.md; nên xử blocker 🔴 trước.

---

## Input / Output

**Input:** analysis từ trạm 1 + skill `qa-planner`.

**Output:** `{qc_dir}/{UC-ID}/{platform}/TEST_PLAN.md` (rủi ro + câu hỏi cho dev).

---

## Các bước xử lý (chi tiết)

1. **Role qa-planner** — nạp `{qc_skills_dir}/qa-planner/`.
2. Đọc analysis + gap → **đánh giá rủi ro** từng vùng.
3. Nêu **câu hỏi cho dev** (điểm chưa rõ để test đúng).
4. **Output** TEST_PLAN.md.

---

## Checkpoint & Gate

- Không gate chặn.

---

## Cơ chế đặc biệt

- **Risk-based** — kế hoạch bám rủi ro, không phủ đều.
- **Câu hỏi cho dev** — kênh QC ↔ Dev có hồ sơ trước khi viết test.

---

## 👓 Góc nhìn tối ưu

- **Trạm mỏng nhất** (~thin orchestrator + skill). Giá trị phụ thuộc chất lượng skill qa-planner.
- **Câu hỏi cho dev** không có cơ chế theo dõi trả lời — dễ rơi. Cân nhắc gắn vào DOC_GAPS/feedback.

---

## Kết nối

**Trước:** [`/qc-analyze`](15-qc-analyze.md) · **Sau:** [`/qc-design-test`](17-qc-design-test.md).
