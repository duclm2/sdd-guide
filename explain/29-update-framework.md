[← /sync](28-sync.md) · [Explain Home](README.md)

# 29 · `/update-framework` — Nâng cấp bản framework (npm)

> **Một câu.** Sync bản npm mới của framework — **ghi đè** file framework (`.agent/commands|steps|hooks|rules|templates|skills|modules`) nhưng **giữ nguyên** nội dung của bạn (`CLAUDE.md`, `project-context.yaml`, `domain-knowledge/`, `.trace/`).

---

## Vấn đề giải quyết

Framework tiến hoá qua npm. Command nâng cấp an toàn: cập nhật cơ chế mới mà **không đụng** cấu hình/tri thức dự án — tách rõ "code framework" vs "nội dung của bạn".

---

## Vị trí & tiền đề

- **Vị trí:** xuyên suốt (vận hành).
- **Tiền đề:** Node.js + network/npm; dự án cài qua `--init` (có `.agent/FRAMEWORK_VERSION`, thiếu → dừng).

---

## Input / Output

**Input:** bản npm mới `@educa-corp/sdd-framework`.

**Output:** `.agent/*` cập nhật; `git diff .agent/` để review + commit.

---

## Các bước xử lý (chi tiết)

1. **Step 0 · Phát hiện trạng thái hiện tại** — version đang dùng (`.agent/FRAMEWORK_VERSION`), mode, danh sách module đã cài.
2. **Step 1 · Kiểm tra version mới nhất** — `npm view … version`; đã mới nhất → dừng; có bản mới → hỏi `Proceed with upgrade? (Y/N)`.
3. **Step 2 · Umbrella Awareness** (umbrella) — framework chỉ sống ở umbrella root; service submodule không cần update riêng.
4. **Step 3 · Pre-flight Git Check** — có thay đổi chưa commit trong `.agent/` hoặc `.claude/commands/` → cảnh báo, hỏi `(Y/N)`, mặc định dừng.
5. **Step 4 · Chạy nâng cấp** — `npx -y @educa-corp/sdd-framework@latest --init --module …` (truyền lại mọi module đã cài); ghi đè file framework + `.claude/commands/`; **KHÔNG** đụng `CLAUDE.md`, `project-context.yaml`, `domain-knowledge/`, `.trace/`.
6. **Step 5 · Review Changes** — `git diff --stat`; tóm tắt lệnh mới / đổi / **bị gỡ** (installer gỡ file framework đã bỏ nếu còn nguyên bản, giữ lại file bạn đã sửa); file bạn sửa trong `.agent/` bị ghi đè → báo nổi bật kèm bản cũ ở `.agent/.overwritten-{version}-{date}/`.
7. **Step 5.5 · Kiểm bố cục spec cần migrate** — chỉ quét, không tự chuyển: BDD phẳng → `--migrate-bdd-platform`; bố cục artifact-type-first hoặc trace phẳng → `--migrate-specs` (dry-run trước, `--apply` sau).

---

## Checkpoint & Gate

- **Không chạy gate** (lệnh bảo trì) — thay bằng hai câu hỏi `(Y/N)` riêng: xác nhận nâng cấp (Step 1) và tiếp tục khi `.agent/` có thay đổi chưa commit (Step 3, mặc định dừng).
- Review diff trước commit.

---

## Cơ chế đặc biệt

- **Ranh giới ghi đè rõ** — framework file bị thay, nội dung dự án được giữ. Đây là điều khiến nâng cấp an toàn.
- **Review diff bắt buộc** — người xác nhận thay đổi framework trước khi commit.

---

## 👓 Góc nhìn tối ưu

- **Ranh giới "giữ vs ghi đè"** là sống còn — nếu người dùng lỡ sửa tay `.agent/commands/*.md` (thay vì `.tmpl`), bản nâng cấp vẫn ghi đè. Từ v0.4.2 installer cứu bản cũ vào `.agent/.overwritten-*/` và report nêu nổi bật, nhưng không có hook chặn việc sửa sai chỗ.
- **Pre-flight git sạch** ngăn mất chỉnh chưa commit — tốt, nhưng có thể chặn người quen làm việc với tree bẩn.
- **Sau update → `/sync`** để lan nội dung — chuỗi phụ thuộc người chạy tiếp.

---

## Kết nối

**Trước:** bất kỳ (vận hành) · **Sau:** review `git diff .agent/` + commit; [`/sync`](28-sync.md) để đồng bộ nội dung dự án.
