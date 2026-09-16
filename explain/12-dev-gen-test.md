[← /review-code](10-review-code.md) · [Explain Home](README.md) · [Next: /dev-run-test →](13-dev-run-test.md)

# 12 · `/dev-gen-test` — Sinh bộ tự-kiểm của Dev

> **Một câu.** Sinh bộ **self-test nhanh** (unit/integration theo platform) bám scenario, gắn `@trace.verifies`, để Dev tự kiểm code vừa sinh — trước QC chính thức.

---

## Vấn đề giải quyết

Sau `/generate-code`, Dev cần một lớp test nhanh để bắt lỗi hiển nhiên tại chỗ. Đây **không** phải bộ test chính thức (đó là QC) — mà là smoke của dev, gắn với cột `dev_selftest`.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Dev Self-Test, sau `/generate-code`.
- **Tiền đề:** code build được + `.feature`.

---

## Input / Output

**Input:** code UC + `.feature` + `active_module`/`platform_type` + stack-profile (test pattern).

**Output:** file test (tag `@trace.verifies={UC-ID}`, `@trace.test_type=unit|integration`) + cập nhật trace + Panel Mirror.

---

## Các bước xử lý (chi tiết)

1. **Service Detection** — route service (umbrella) / single.
2. **CHECKPOINT — Test Plan** — 🛑 trình kế hoạch test (loại test, scenario phủ) → chờ Y.
3. **Generate** — sinh test theo test pattern của stack (từ stack-profile): unit test cho logic, integration cho boundary; mỗi test tag `@trace.verifies` + `@trace.test_type`.
4. **Checklist** — tự kiểm bộ test đủ phủ scenario.
5. **Write Trace State** — cập nhật `.tsv` (`test_count`).
6. **Refresh Panel Mirror** — làm mới Living Docs local (umbrella).

---

## Checkpoint & Gate

- 🛑 CHECKPOINT Test Plan trước generate.
- Không gate chặn downstream.

---

## Cơ chế đặc biệt

- **Theo test pattern của stack** — đọc stack-profile thay vì hardcode → cùng lệnh chạy cho java-spring/react/flutter…
- **Tag test_type** phân biệt unit/integration.
- **Orthogonal với QC** — bộ này phục vụ `dev_selftest`, không phải `qc_status`.

---

## 👓 Góc nhìn tối ưu

- **Ranh giới với QC** — dev-gen-test sinh test "nhanh", qc-design-test sinh test-case "chính thức". Có thể trùng công sức viết test 2 lần. Đáng xem có tái dùng được không.
- **Phụ thuộc stack-profile `test_types`** — stack thiếu khai báo → test kém định hình.
- **CHECKPOINT cho mọi lần** — với thay đổi nhỏ có thể là ma sát; cân nhắc short-circuit.

---

## Kết nối

**Trước:** [`/generate-code`](09-generate-code.md) · **Sau:** [`/dev-run-test`](13-dev-run-test.md).
