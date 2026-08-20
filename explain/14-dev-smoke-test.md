[← /dev-run-test](13-dev-run-test.md) · [Explain Home](README.md) · [Next: /qc-analyze →](15-qc-analyze.md)

# 14 · `/dev-smoke-test` — Thử tại chỗ trên service/app đang chạy

> **Một câu.** Kiểm tra nhanh feature **trên service/app đang chạy thật** (health endpoint, curl, chạy app trên simulator) — smoke cuối của dev trước khi giao.

---

## Vấn đề giải quyết

Test đơn vị pass ≠ chạy thật OK. `/dev-smoke-test` xác nhận feature hoạt động trên runtime thật (server lên, endpoint trả 200, app chạy không crash) — lớp kiểm cuối trước PR/QC.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Dev Self-Test (cuối), sau `/dev-run-test`.
- **Tiền đề:** service/app khởi động được (`conventions.service_run`).

---

## Input / Output

**Input:** service/app đang chạy + `.feature` + `platform_type`.

**Output:** kết quả smoke tại chỗ (báo cáo), chẩn crash nếu có.

---

## Các bước xử lý (chi tiết)

Rẽ nhánh theo `platform_type`:

| platform_type | Việc |
|---------------|------|
| **backend** | Thử health endpoint; gọi GET/POST thật (curl); hoặc pytest smoke marker |
| **web-frontend** | curl kiểm 200 (Linux/mac/Windows); nếu không → khởi động bằng `service_run`; chạy Playwright/Cypress scenario gắn UC |
| **mobile** | Chạy app trên simulator (Flutter/React Native/Android Compose/iOS SwiftUI); đọc crash log nếu lỗi |

---

## Checkpoint & Gate

- Không gate chặn — smoke tư vấn. Pass → tạo PR.

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

**Trước:** [`/dev-run-test`](13-dev-run-test.md) · **Sau:** tạo PR, hoặc chuyển QC chính thức → [`/qc-analyze`](15-qc-analyze.md).
