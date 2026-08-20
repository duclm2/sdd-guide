[← /debug](24-debug.md) · [Explain Home](README.md) · [Next: /propose-scenario →](26-propose-scenario.md)

# 25 · `/report-bug` — File bug có trace-spec (cho Tester & QC)

> **Một câu.** Ghi một bug **gắn spec** (AC bị vi phạm, layer khả nghi), backfill trace, và handoff để PO/Dev thực sự thấy.

---

## Vấn đề giải quyết

Bug không gắn spec khó truy vết & regression. Command ép bug được neo vào UC/SC/AC cụ thể, phân loại layer khả nghi, và nổi lên đúng người — làm input chất lượng cho `/fix-bug`.

---

## Vị trí & tiền đề

- **Vị trí:** xuyên suốt (kênh feedback tester/QC), gồm cả product-gap từ `/qc-*`.

---

## Input / Output

**Input:** mô tả bug + UC/platform + spec context.

**Output:** `{bug_reports_dir}/{BUG-ID}.md` (spec-anchored) + backfill trace + handoff.

---

## Các bước xử lý (chi tiết)

1. **Step 1 · Phân giải Spec Context** — định vị UC/SC/PRD liên quan.
2. **Step 2 · Thu thập chi tiết Bug** — hiện tượng, bước tái hiện, môi trường.
3. **Step 3 · Xác định AC bị vi phạm** — bug này vi phạm AC nào (neo nghiệp vụ).
4. **Step 4 · Phân loại Layer khả nghi (BUG_FLOW)** — đoán layer gây lỗi (FE/BE/data…).
5. **Step 5 · Ghi Report** — file `{BUG-ID}.md`.
6. **Step 5.5 · Backfill trace** — link pending-view (nối bug vào trace).
7. **Step 6 · Handoff** — nổi lên để PO/Dev thấy (qua `/sync`).

---

## Checkpoint & Gate

- Không gate — kênh có hồ sơ.

---

## Cơ chế đặc biệt

- **Spec-anchored** — bug neo AC/SC → `/fix-bug` có ngữ cảnh, regression đúng chỗ.
- **BUG_FLOW layer classification** — hướng dev tới layer khả nghi.
- **Backfill trace pending-view** — bug hiện trong dashboard trace.

---

## 👓 Góc nhìn tối ưu

- **Product-gap từ `/qc-report` → `/report-bug`** không tự động — nối tay. Đáng tích hợp để gap thành bug spec-anchored liền mạch.
- **Handoff qua `/sync`** — nếu không chạy sync, feedback nằm im. Phụ thuộc nhịp vận hành.
- **Phân loại layer là phỏng đoán** — có thể sai; `/debug`/`/fix-bug` xác nhận lại.

---

## Kết nối

**Trước:** QC/test phát hiện lỗi · **Sau:** [`/fix-bug {BUG-ID}`](23-fix-bug.md); thiếu coverage → [`/propose-scenario`](26-propose-scenario.md); nổi lên qua [`/sync`](28-sync.md).
