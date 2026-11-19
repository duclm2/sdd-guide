[← /validate-traces](21-validate-traces.md) · [Explain Home](README.md) · [Next: /fix-bug →](23-fix-bug.md)

# 22 · `/generate-spec-manifest` — Mục lục spec cho agent ngoài

> **Một câu.** Sinh một **manifest** (mục lục máy-đọc) của mọi spec trong dự án, để agent/công cụ bên ngoài định vị PRD/BDD/tech-docs mà không phải quét cả repo.

---

## Vấn đề giải quyết

Agent ngoài (hoặc lệnh khác) cần biết "có những spec nào, ở đâu, trạng thái gì" mà không đọc toàn bộ. Manifest là index tự sinh, luôn cập nhật khi chạy lại.

---

## Vị trí & tiền đề

- **Vị trí:** xuyên suốt (thường sau `/sync`).
- **Đặc biệt:** file manifest **gitignored** (auto-generated, không commit).

---

## Input / Output

**Cú pháp:** `/generate-spec-manifest [domain]` — không có target file; domain filter tuỳ chọn.

**Input:** cây spec (`specs/{domain}/{prd-slug}/…`) + product-definition (PDD); umbrella mode quét `{setup.spec_source}` và map `services`.

**Output:** `spec-manifest.yaml` (auto-generated, DO NOT edit; regenerate bằng chính lệnh này).

---

## Các bước xử lý (chi tiết)

1. Xác định scan root theo mode (umbrella / single-service), quét PRD theo bố cục feature-package — đọc `PRD ID`, `Domain`, `Status`, `Version` từ metadata; bỏ file không xác định được TICKET-ID.
2. Khớp PDD theo TICKET-ID; khớp BDD và tech-doc **theo từng service** (glob tên file, fallback `@trace.prd`). Service không có file thì bỏ key, không ghi `null`.
3. Ghi `spec-manifest.yaml` (ghi đè, sort theo TICKET-ID) với header "Auto-generated — DO NOT edit".
4. Đảm bảo `spec-manifest.yaml` có trong `.gitignore` (thiếu thì append). Report liệt kê PRD chưa có BDD / tech-doc khớp.

---

## Checkpoint & Gate

- **CHECKPOINT: chặn thường** (`--yes` bỏ qua được) — chỉ mục suy 100% từ cây `specs/`, ghi đè không mất gì.

---

## Cơ chế đặc biệt

- **Máy-đọc, gitignored** — index tái sinh, không phải nguồn chân lý (spec mới là nguồn).
- **Cho agent ngoài** — điểm tích hợp với công cụ/agent khác.

---

## 👓 Góc nhìn tối ưu

- **Phải chạy lại để cập nhật** — dễ stale nếu quên. `/sync` Step 6 làm mới khi file đã tồn tại hoặc có `setup.spec_source`; lần đầu (chưa có file, không có `spec_source`) phải chạy tay.
- **Trùng vai với `/validate-traces`?** — validate-traces là trạng thái coverage; manifest là index vị trí. Khác mục đích nhưng cùng "bản đồ spec".

---

## Kết nối

**Trước:** [`/sync`](28-sync.md) hoặc bất kỳ · **Sau:** agent ngoài dùng manifest để định vị spec.
