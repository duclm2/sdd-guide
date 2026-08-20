[← /sync](28-sync.md) · [Explain Home](README.md)

# 29 · `/update-framework` — Nâng cấp bản framework (npm)

> **Một câu.** Sync bản npm mới của framework — **ghi đè** file framework (`.agent/commands|steps|hooks|rules|templates|skills|modules`) nhưng **giữ nguyên** nội dung của bạn (`CLAUDE.md`, `project-context.yaml`, `domain-knowledge/`, `.trace/`).

---

## Vấn đề giải quyết

Framework tiến hoá qua npm. Command nâng cấp an toàn: cập nhật cơ chế mới mà **không đụng** cấu hình/tri thức dự án — tách rõ "code framework" vs "nội dung của bạn".

---

## Vị trí & tiền đề

- **Vị trí:** xuyên suốt (vận hành).
- **Tiền đề:** Node.js; git sạch (pre-flight check).

---

## Input / Output

**Input:** bản npm mới `@educa-corp/sdd-framework`.

**Output:** `.agent/*` cập nhật; `git diff .agent/` để review + commit.

---

## Các bước xử lý (chi tiết)

1. **Step 0 · Phát hiện trạng thái hiện tại** — version đang dùng, mode.
2. **Step 1 · Kiểm tra version mới nhất** — `npm view … version`.
3. **Step 2 · Umbrella Awareness** (umbrella) — lưu ý nhiều submodule.
4. **Step 3 · Pre-flight Git Check** — đảm bảo working tree sạch trước khi ghi đè.
5. **Step 4 · Chạy nâng cấp** — ghi đè file framework; **KHÔNG** đụng `CLAUDE.md`, `project-context.yaml`, `domain-knowledge/`, `.trace/`.
6. **Step 5 · Review Changes** — `git diff .agent/` để người xem trước khi commit.

---

## Checkpoint & Gate

- Pre-flight git check (working tree sạch).
- Review diff trước commit.

---

## Cơ chế đặc biệt

- **Ranh giới ghi đè rõ** — framework file bị thay, nội dung dự án được giữ. Đây là điều khiến nâng cấp an toàn.
- **Review diff bắt buộc** — người xác nhận thay đổi framework trước khi commit.

---

## 👓 Góc nhìn tối ưu

- **Ranh giới "giữ vs ghi đè"** là sống còn — nếu người dùng lỡ sửa tay `.agent/commands/*.md` (thay vì `.tmpl`), thay đổi mất khi update. Quy ước này phụ thuộc kỷ luật + memory, không có hook chặn.
- **Pre-flight git sạch** ngăn mất chỉnh chưa commit — tốt, nhưng có thể chặn người quen làm việc với tree bẩn.
- **Sau update → `/sync`** để lan nội dung — chuỗi phụ thuộc người chạy tiếp.

---

## Kết nối

**Trước:** bất kỳ (vận hành) · **Sau:** review `git diff .agent/` + commit; [`/sync`](28-sync.md) để đồng bộ nội dung dự án.
