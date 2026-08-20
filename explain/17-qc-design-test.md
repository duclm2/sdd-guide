[← /qc-plan](16-qc-plan.md) · [Explain Home](README.md) · [Next: /qc-review →](18-qc-review.md)

# 17 · `/qc-design-test` — Trạm 3: Thiết kế test case (Markdown)

> **Một câu.** Thiết kế **test case dạng Markdown** (`.Test.md`) từ plan — chưa sinh Python; script đến sau ở `/qc-run-test`.

---

## Vấn đề giải quyết

Tách "thiết kế test case" (con người đọc/review được) khỏi "code test" (máy chạy). `.Test.md` là bản thiết kế mà `/qc-review` duyệt và `/qc-run-test` biến thành script.

---

## Vị trí & tiền đề

- **Vị trí:** Phase QC (trạm 3), sau `/qc-plan`.

---

## Input / Output

**Input:** TEST_PLAN.md + skill `qa-designer` (chọn layer, nạp MỘT file).

**Output:** `{qc_dir}/{UC-ID}/{platform}/test-cases/*.Test.md` (trace mapping bắt buộc).

---

## Các bước xử lý (chi tiết)

1. **Role qa-designer** — chọn layer, nạp một file skill (không nạp cả bộ → tiết kiệm context).
2. **Conventions** — theo quy ước test-case của QC team.
3. **Trace mapping (bắt buộc)** — mỗi test case ↔ SC.
4. **Output** `.Test.md`.

---

## Checkpoint & Gate

- Không gate ở đây; gate là `/qc-review` kế tiếp.

---

## Cơ chế đặc biệt

- **Markdown-first** — test case là tài liệu review được trước khi thành code.
- **Nạp MỘT file skill theo layer** — tối ưu context (chỉ layer cần).
- **Trace mapping** — giữ test case gắn SC để về sau ghi `qc_status` đúng.

---

## 👓 Góc nhìn tối ưu

- **Tách design ↔ script** thêm một trạm nhưng cho phép review sớm — đánh đổi giữa số bước và chất lượng gate.
- **Phụ thuộc convention QC team** — cần đồng bộ khi convention đổi.

---

## Kết nối

**Trước:** [`/qc-plan`](16-qc-plan.md) · **Sau:** [`/qc-review`](18-qc-review.md) (review test case).
