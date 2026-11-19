[← /dev-run-test](13-dev-run-test.md) · [Explain Home](README.md) · [Next: /qc-analyze →](15-qc-analyze.md)

# 14 · `/dev-smoke-test` — Thử tại chỗ trên service/app đang chạy

> **Một câu.** Kiểm tra nhanh feature **trên service/app đang chạy thật** (health endpoint, curl, chạy app trên simulator) — smoke cuối của dev trước khi giao.

---

## Vấn đề giải quyết

Test đơn vị pass ≠ chạy thật OK. `/dev-smoke-test` xác nhận feature hoạt động trên runtime thật (server lên, endpoint trả 200, app chạy không crash) — lớp kiểm cuối trước PR/QC.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Dev Self-Test (cuối), sau `/review-code` (bảng Next: `/review-code` → `/dev-smoke-test` hoặc tạo PR).
- **Tiền đề:** service/app **đang chạy sẵn** (chưa chạy thì lệnh chỉ nhắc khởi động bằng `conventions.service_run`).

---

## Input / Output

**Input:** service/app đang chạy + `.feature` + `platform_type`.

**Output:** kết quả smoke tại chỗ (báo cáo), chẩn crash nếu có. **Không ghi file nào** (`Output Artifacts: none`).

---

## Các bước xử lý (chi tiết)

Rẽ nhánh theo `platform_type`:

| platform_type | Việc |
|---------------|------|
| **backend** | Thử health endpoint → tìm endpoint qua `@trace.implements={UC-ID}` → hỏi auth token (dán / token dev / chỉ endpoint public) → gọi GET/POST thật (curl). `context-engineering`: gọi thẳng prompt function hoặc `{test_command} -m smoke`, rồi so output với schema |
| **web-frontend** | curl kiểm 200 (Linux/mac/Windows); nếu không → khởi động bằng `service_run`; chạy Playwright/Cypress scenario gắn UC |
| **mobile** | Chạy app trên simulator (Flutter/React Native/Android Compose/iOS SwiftUI); đọc crash log nếu lỗi |

---

## Checkpoint & Gate

- Nhãn khai **không chặn** — lệnh không ghi file nào, nên gate không hỏi CHECKPOINT. Smoke tư vấn: pass → tạo PR; lỗi → `/debug` hoặc `/fix-bug`.

---

## Cơ chế đặc biệt

- **Chạy trên runtime thật** — khác dev-run-test (chạy test suite); đây là "bấm thử" thực tế.
- **Rẽ nhánh 3 platform_type** với lệnh cụ thể cho từng stack + đọc crash log mobile.
- **Bắc cầu sang QC E2E** — web dùng chính Playwright/Cypress mà QC sẽ dùng.

---

## 👓 Góc nhìn tối ưu

- **Chồng lấn QC E2E** — web smoke đã chạy Playwright scenario; QC cũng Playwright. Ranh giới: dev-smoke = nhanh/thủ công, QC = đầy đủ/evidence. Đáng làm rõ.
- **Phụ thuộc service chạy được** — nếu môi trường local chưa dựng, smoke không chạy → giá trị giảm.
- **Không ghi trace** (khác dev-run-test) — kết quả smoke không lưu trạng thái; chỉ báo cáo. Cân nhắc có nên lưu.
- **Ba nhánh platform** với nhiều biến thể lệnh — chi phí bảo trì theo số stack.

---

## Kết nối

**Trước:** [`/review-code`](10-review-code.md) (sau [`/dev-run-test`](13-dev-run-test.md)) · **Sau:** tạo PR và link tới ticket; lỗi → [`/debug`](24-debug.md) / [`/fix-bug`](23-fix-bug.md). Làn QC không bắt đầu ở đây — nó đã chạy song song từ sau `/review-tech-docs`, vào từ [`/qc-analyze`](15-qc-analyze.md).
