[← /fix-bug](23-fix-bug.md) · [Explain Home](README.md) · [Next: /report-bug →](25-report-bug.md)

# 24 · `/debug` — Phân tích debug nhanh (không sửa code)

> **Một câu.** Phân tích lỗi (stack trace, error pattern, test failure) và đề xuất hướng sửa — **không sửa code**, chỉ chẩn đoán.

---

## Vấn đề giải quyết

Trước khi sửa, cần hiểu lỗi. `/debug` phân tích nhanh nguyên nhân + vị trí + hướng fix, tách "hiểu lỗi" khỏi "sửa lỗi" (`/fix-bug`).

---

## Vị trí & tiền đề

- **Vị trí:** xuyên suốt.
- **Đặc biệt:** không sửa code, không commit, không cần ticket. Thứ duy nhất lệnh có thể ghi là project lesson (`{lessons_path}`), và chỉ sau câu hỏi `(Y/N)`.

---

## Input / Output

**Input:** mô tả lỗi / stack trace / test failure + `platform_type` + project lessons.

**Output:** báo cáo — Error · Root Cause · Location · Suggested Fix · Related Rule · Next Step. Không ghi file nào, trừ khi người dùng đồng ý ghi lesson.

---

## Các bước xử lý (chi tiết)

0. **Step 1 · Chọn tình huống** — menu 4 lựa chọn: (1) đã có stack trace/log → dán · (2) cần reproduce → lệnh hiện `conventions.service_run` · (3) test đang fail → hiện `conventions.test_command` rồi chờ dán output · (4) câu hỏi code → trả lời thẳng theo context đã nạp. Thiếu `service_run`/`test_command` → cảnh báo kèm cách cấu hình.
1. **Stack Trace Analysis** — đọc stack trace từ **dưới lên** (`Caused by:` là root cause thật), định vị.
2. **Common Error Patterns** — đối chiếu pattern lỗi quen thuộc theo platform.
3. **Test Failure Analysis** — nếu là test fail, phân tích nguyên nhân.
4. **Output** cấu trúc: Error / Root Cause / Location / Suggested Fix / Related Rule (trỏ mục CLAUDE.md liên quan).
5. **Đề xuất ghi Lesson** *(tuỳ chọn)* — root cause là lỗi AI sinh code có thể lặp lại → hỏi `(Y/N)`, `Y` thì chạy `capture-lesson` với `source=/debug`.

---

## Checkpoint & Gate

- **Không chặn** — *không phải vì read-only*: lệnh có thể ghi lesson, nhưng chỉ sau câu hỏi `(Y/N)`.

---

## Cơ chế đặc biệt

- **Chẩn đoán theo platform** (`platform_type`, bảng pattern riêng cho backend · context-engineering · web-frontend · mobile) + đối chiếu **project lessons** (nạp ở context-loader).
- **Related Rule** — nối lỗi với rule đã có trong CLAUDE.md; lỗi AI lặp lại thì có đường ghi thành lesson mới.

---

## 👓 Góc nhìn tối ưu

- **Read-only tách khỏi fix** — sạch về vai, nhưng người dùng phải chuyển sang `/fix-bug` thủ công. Có thể chuyển giao mượt hơn (truyền root cause sang fix-bug).
- **Common Error Patterns** — độ hữu ích phụ thuộc pattern có bám stack thực tế của dự án không.

---

## Kết nối

**Trước:** lỗi bất kỳ / [`/dev-run-test`](13-dev-run-test.md) (fail) · **Sau:** cần sửa → [`/fix-bug`](23-fix-bug.md).
