[← /fix-bug](23-fix-bug.md) · [Explain Home](README.md) · [Next: /report-bug →](25-report-bug.md)

# 24 · `/debug` — Phân tích debug nhanh (read-only)

> **Một câu.** Phân tích lỗi (stack trace, error pattern, test failure) và đề xuất hướng sửa — **không sửa code**, chỉ chẩn đoán.

---

## Vấn đề giải quyết

Trước khi sửa, cần hiểu lỗi. `/debug` phân tích nhanh nguyên nhân + vị trí + hướng fix, tách "hiểu lỗi" khỏi "sửa lỗi" (`/fix-bug`).

---

## Vị trí & tiền đề

- **Vị trí:** xuyên suốt.
- **Đặc biệt:** **read-only** — không sửa, không commit.

---

## Input / Output

**Input:** mô tả lỗi / stack trace / test failure + `platform_type` + project lessons.

**Output:** báo cáo — Error · Root Cause · Location · Suggested Fix · Related Rule (`Output Artifacts: none (read-only)`).

---

## Các bước xử lý (chi tiết)

1. **Stack Trace Analysis** — đọc stack trace, định vị.
2. **Common Error Patterns** — đối chiếu pattern lỗi quen thuộc theo platform.
3. **Test Failure Analysis** — nếu là test fail, phân tích nguyên nhân.
4. **Output** cấu trúc: Error / Root Cause / Location / Suggested Fix / Related Rule (trỏ rule/lesson liên quan).

---

## Checkpoint & Gate

- Read-only, không gate.

---

## Cơ chế đặc biệt

- **Chẩn đoán theo platform** (`platform_type`) + đối chiếu **project lessons**.
- **Related Rule** — nối lỗi với rule/lesson đã có → học từ lịch sử.

---

## 👓 Góc nhìn tối ưu

- **Read-only tách khỏi fix** — sạch về vai, nhưng người dùng phải chuyển sang `/fix-bug` thủ công. Có thể chuyển giao mượt hơn (truyền root cause sang fix-bug).
- **Common Error Patterns** — độ hữu ích phụ thuộc pattern có bám stack thực tế của dự án không.

---

## Kết nối

**Trước:** lỗi bất kỳ / [`/dev-run-test`](13-dev-run-test.md) (fail) · **Sau:** cần sửa → [`/fix-bug`](23-fix-bug.md).
