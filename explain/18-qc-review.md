[← /qc-design-test](17-qc-design-test.md) · [Explain Home](README.md) · [Next: /qc-run-test →](19-qc-run-test.md)

# 18 · `/qc-review` — Trạm 4: Cổng review hai chiều (test case & script)

> **Một câu.** Gate chất lượng của QC: review **test case** (sau design-test) VÀ **script** (sau run-test) — chạy ở hai thời điểm trong dây chuyền.

---

## Vấn đề giải quyết

Không chạy test kém. Trạm này chặn: (1) test case chưa đủ tốt trước khi biến thành script; (2) script chưa đúng trước khi coi kết quả là chính thức.

---

## Vị trí & tiền đề

- **Vị trí:** Phase QC (trạm 4) — **hai lần**: sau `/qc-design-test` (review case) và sau `/qc-run-test` (review script).

---

## Input / Output

**Input:** `.Test.md` (case) hoặc script Python (script) + skill `qa-reviewer`.

**Output:** verdict APPROVED / NEEDS_FIX + findings review.

---

## Các bước xử lý (chi tiết)

1. **Role qa-reviewer** — nạp `{qc_skills_dir}/qa-reviewer/`.
2. **Review focus** — kiểm test case (đủ phủ, đúng SC) hoặc script (đúng logic, selector, không fake-pass).
3. Verdict: APPROVED → đi tiếp; NEEDS_FIX → sửa rồi review lại.

---

## Checkpoint & Gate

- 🛑 **Cổng hai chiều** — chặn `/qc-run-test` (nếu case chưa duyệt) và chặn tạo PR (nếu script chưa duyệt).

---

## Cơ chế đặc biệt

- **Một lệnh, hai vai** — review case và review script dùng chung skill, khác focus.
- **Gate thật của QC** — điểm HITL chính trong dây chuyền tự động.

---

## 👓 Góc nhìn tối ưu

- **Chạy 2 lần cùng một lệnh** — người dùng phải nhớ gọi đúng thời điểm. Footer "Next" hướng dẫn, nhưng dễ nhầm. Cân nhắc tách rõ hoặc auto-detect giai đoạn.
- **Verdict thủ công** — phụ thuộc reviewer; không có findings-file có mã như review-context. Đáng xem có nên chuẩn hoá.

---

## Kết nối

**Trước:** [`/qc-design-test`](17-qc-design-test.md) (case) / [`/qc-run-test`](19-qc-run-test.md) (script) · **Sau:** case APPROVED → [`/qc-run-test`](19-qc-run-test.md); script APPROVED → [`/qc-report`](20-qc-report.md) / PR.
