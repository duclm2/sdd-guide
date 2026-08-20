[← /generate-architecture](00b-generate-architecture.md) · [Explain Home](README.md) · [Next: /generate-prd →](02-generate-prd.md)

# 01 · `/define-product` — Khám phá tính năng (Q&A 8 Phase)

> **Một câu.** Dẫn PO qua một buổi hỏi-đáp **8 phase có checkpoint** để biến ý tưởng bằng lời thành một **product-definition** có cấu trúc — nền cho PRD.

---

## Vấn đề giải quyết

Đây là **thượng nguồn**. Sai ở đây nhân lên cấp số nhân xuống toàn pipeline. Command ép PO diễn đạt intent **một lần, có cấu trúc, chốt từng phase** — để người lẫn AI hiểu giống nhau, không nhảy thẳng ý-tưởng → code.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Discovery (bước 1).
- **Tiền đề:** đã `/setup-ai-first`; có ý tưởng bằng lời (+ tuỳ chọn Figma).
- **Gate ra:** không gate cứng — output là `product-definition` (chưa phải PRD).

---

## Input / Output

**Input:** ý tưởng natural language + context nền (CLAUDE.md, domain-knowledge nạp qua context-loader).

**Output:** `{product_definitions_dir}/{TICKET-ID}-{slug}.md` theo `product-definition.template.md`, có **Metadata với `Completed Phase` + `Status`** để resume.

> ⭐ **`slug` sinh ở đây là định danh feature-package dùng xuyên suốt** — PRD/BDD/tech-docs/design-spec/trace đều kế thừa nguyên văn. Sinh một lần, không tái sinh.

---

## Các bước xử lý (chi tiết)

Sau Gate + Context-Loader chung, chạy **Business Language Guard** (giữ giọng nghiệp vụ) rồi 8 phase:

| Phase | Tên | Việc | Checkpoint |
|-------|-----|------|:----------:|
| **0** | Knowledge Sync | AI **tự** quét dự án: khái niệm liên quan, feature bị ảnh hưởng, rule có sẵn, **chuẩn hoá thuật ngữ** (map input PO → business-dictionary; phát hiện NEW TERM lặp ≥2 lần → dồn hỏi ở Phase 3) | — (AI tự) |
| **1** | Feature Definition | Hỏi **lần lượt từng câu**: Context · Problem · Goal · Actors · In/Out Scope · User Story · **Phụ thuộc liên service** | 🛑 1 |
| **2** | User Flow | Entry point · Flow steps (bảng) · **Màn hình & thành phần chính** (nguồn cho Wireframe + coverage BDD) · Exit point · Edge cases | 🛑 2 |
| **3** | Clarification Log | AI xác định gap, hỏi follow-up nhiều vòng. **BLOCK: còn "Mục chưa giải quyết" → không sang Phase 4** | 🛑 3 |
| **4** | Business Rules | Suy từ Phase 1–3: mỗi BR = system MUST/MUST NOT + điều kiện | 🛑 4 |
| **5** | Business Logic | Map mỗi BR → logic nghiệp vụ (rẽ nhánh/công thức) — KHÔNG mô tả thay đổi dữ liệu/UI | 🛑 5 |
| **6** | Acceptance Criteria | Suy từ BR + User Story; mỗi AC testable (pass/fail rõ) | 🛑 6 |
| **7** | Validation Report | AI tự sinh **ma trận độ phủ** (Flow action × Rule/Logic/AC), phát hiện xung đột & gap | — |

Mỗi checkpoint: AI tóm tắt → chờ PO `✅ xác nhận: Có` → mới sang phase sau. **Cập nhật `Completed Phase` sau mỗi checkpoint** → resume được nếu ngắt giữa chừng.

---

## Checkpoint & Gate

- 🔴 **Mật độ HITL cao nhất toàn pipeline** — 6 checkpoint tường minh (Phase 1–6) + BLOCK ở Phase 3.
- Hỏi **từng câu một** (không dồn), có ví dụ gợi ý khi PO lúng túng.

---

## Cơ chế đặc biệt

- **Phase 0 tách khỏi Q&A** — AI tự làm phần "bối cảnh hệ thống" (không cần PO), chỉ dồn câu hỏi thật vào các phase sau → không phá trải nghiệm.
- **NEW TERM detection** — thuật ngữ mới lặp ≥2 lần được phát hiện ở Phase 0 nhưng **hỏi ở Phase 3**, và có thể cập nhật ngược `business-dictionary.md`.
- **Altitude enforcement** — Phase 4/5/6 tách rõ BR (quy tắc) vs Logic (cơ chế nghiệp vụ) vs AC (nghiệm thu); Business Language Guard chặn kỹ thuật lọt vào.
- **Resume qua Metadata** — `Completed Phase` cho phép ngắt/tiếp buổi khác.

---

## 👓 Góc nhìn tối ưu

- **Chi phí HITL cao** — 6 checkpoint tuần tự, hỏi từng câu, có thể dài với feature nhỏ. Đáng cân nhắc "fast mode" cho feature đơn giản (gộp phase, short-circuit khi AI đủ tự tin) — hiện chưa có.
- **Phase 3 BLOCK** là điểm dễ kẹt nếu AI đặt câu hỏi không hội tụ. Chất lượng câu hỏi phụ thuộc Phase 0 quét dự án tốt tới đâu.
- **Trùng "8 chặng" vs "8 phase"** — tài liệu concept từng nói "8 chặng"; thực tế là Phase 0–7. Cần đồng bộ để review không nhầm.
- **Ma trận Phase 7** là một dạng self-review sớm — có thể tái dùng ý này ở downstream.

---

## Kết nối

**Trước:** [`/setup-ai-first`](00-setup-ai-first.md) · **Sau:** [`/generate-prd {product-definition-file}`](02-generate-prd.md).
