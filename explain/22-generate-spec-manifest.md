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

**Input:** cây spec (`specs/{domain}/{prd-slug}/…`).

**Output:** `spec-manifest.yaml` (auto-generated, DO NOT edit; regenerate bằng chính lệnh này).

---

## Các bước xử lý (chi tiết)

1. Quét `specs/` theo bố cục feature-package.
2. Với mỗi feature: ghi domain, prd-slug, TICKET-ID, các artifact (PRD/BDD/tech-docs/design-spec), trạng thái.
3. Ghi `spec-manifest.yaml` với header "Auto-generated — do not commit".

---

## Checkpoint & Gate

- Không gate.

---

## Cơ chế đặc biệt

- **Máy-đọc, gitignored** — index tái sinh, không phải nguồn chân lý (spec mới là nguồn).
- **Cho agent ngoài** — điểm tích hợp với công cụ/agent khác.

---

## 👓 Góc nhìn tối ưu

- **Phải chạy lại để cập nhật** — dễ stale nếu quên. `/sync` Step 6 làm mới cho umbrella; single-service phải nhớ chạy tay.
- **Trùng vai với `/validate-traces`?** — validate-traces là trạng thái coverage; manifest là index vị trí. Khác mục đích nhưng cùng "bản đồ spec".

---

## Kết nối

**Trước:** [`/sync`](28-sync.md) hoặc bất kỳ · **Sau:** agent ngoài dùng manifest để định vị spec.
