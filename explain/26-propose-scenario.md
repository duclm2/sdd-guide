[← /report-bug](25-report-bug.md) · [Explain Home](README.md) · [Next: /learn →](27-learn.md)

# 26 · `/propose-scenario` — Đề xuất BDD scenario mới (cho Tester & QC)

> **Một câu.** Tester đề xuất một scenario còn thiếu; command quyết đây là **scenario mới** (draft) hay **thay đổi PRD** (change request), rồi ghi proposal để PO/Dev duyệt.

---

## Vấn đề giải quyết

Tester thấy coverage thủng nhưng không được sửa spec trực tiếp. Command cho họ kênh **đề xuất có hồ sơ**: nếu là scenario mới trong scope → draft; nếu chạm yêu cầu nghiệp vụ → PRD change request.

---

## Vị trí & tiền đề

- **Vị trí:** xuyên suốt (kênh feedback tester/QC).

---

## Input / Output

**Input:** UC + platform + mô tả scenario thiếu.

**Output:** `{bdd_proposals_dir}/…` (Case A: scenario draft) hoặc PRD change request (Case B).

---

## Các bước xử lý (chi tiết)

1. **Step 1 · Phân giải UC + Platform.**
2. **Step 2 · Quyết định Coverage (CRITICAL)** — phân loại:
   - **Case A** — scenario mới **trong scope** UC hiện tại → draft scenario.
   - **Case B** — đòi hỏi **thay đổi yêu cầu nghiệp vụ** → **PRD Change Request** (không tự draft, đẩy về PO).
3. **Step 3 · Draft Scenario** (chỉ Case A) — viết Gherkin đề xuất.
4. **Step 4 · Ghi Proposal.**
5. **Step 5 · Handoff** — để PO/Dev thấy; `/generate-bdd` có thể incorporate proposal `accepted`.

---

## Checkpoint & Gate

- Không gate — đề xuất, PO/Dev quyết.

---

## Cơ chế đặc biệt

- **Phân biệt scenario-gap vs PRD-change** — giữ ranh giới ai được đổi gì (tester không tự đổi yêu cầu nghiệp vụ).
- **Proposal `accepted` chảy ngược vào `/generate-bdd`** — khép vòng.

---

## 👓 Góc nhìn tối ưu

- **Quyết định Case A/B là điểm phán đoán** — sai hướng thì hoặc phình scope BDD hoặc bỏ sót đổi PRD. Tiêu chí rõ quan trọng.
- **Nối với `/generate-bdd` incorporate** phụ thuộc trạng thái `accepted` được cập nhật — quy trình duyệt cần rõ.

---

## Kết nối

**Trước:** tester thấy coverage thiếu · **Sau:** PO/Dev review proposal.

| Case | Đi đâu | Ai lấy ra |
|---|---|---|
| **A** — thiếu scenario cho AC đã có | `feedback/bdd-proposals/` | [`/generate-bdd`](06-generate-bdd.md) quét thư mục **mỗi lần chạy**, chèn khi `Status: accepted` |
| **B** — requirement MỚI | `feedback/prd-change-requests/` | [`/extend-prd`](02b-extend-prd.md) — **không** phải `/refine-prd` (lệnh đó không thêm được AC mới) |

> Case B **không tự vào BDD được**: scenario chưa có AC để trace tới. Nó chỉ xuất hiện sau khi PO đưa requirement vào PRD rồi chạy lại `/generate-bdd`. Và vì chưa có AC, **không cờ trace nào bắt được** thiếu sót này — theo mọi thước đo coverage thì hành vi đó *không tồn tại*. Đó là lý do [`/validate-traces`](21-validate-traces.md) Step 7b phải đếm và nhắc lại kèm số ngày chờ.
