[← Setup](00-setup.md) · [Pipeline Steps](README.md) · [Next: Specification →](02-specification.md)

# Bước 1 · Discovery — Khám phá tính năng (Feature Discovery)

> **Tóm tắt.** Biến một **ý tưởng bằng lời** thành một **khung intent có cấu trúc** thông qua Q&A 8 chặng, PO chốt từng chặng — *trước khi* viết PRD.
> **Command:** `/define-product`

| | |
|---|---|
| **Giai đoạn** | Discovery (đầu pipeline) |
| **Owner** | 👤 Product Owner / BA |
| **Đầu vào** | Ý tưởng bằng lời (+ tuỳ chọn Figma/wireframe) |
| **Đầu ra** | `product-definition/*.md` — khung intent đã duyệt từng chặng |
| **HITL** | 🔴 **Cao nhất** — chốt mỗi chặng |

---

## Mục đích (Purpose)

Đây là **thượng nguồn của cả pipeline**. Sai ở đây thì mọi thứ downstream (PRD → BDD → code → test) đều sai theo cấp số nhân. Bước Discovery ép PO diễn đạt intent **một lần, có cấu trúc** để:

- Cả người lẫn AI hiểu **giống nhau** ngay từ đầu.
- Không nhảy thẳng từ "ý tưởng" sang "code" bỏ qua tư duy về user/flow/rule/edge-case.
- Tạo **nền tảng ổn định** để `/generate-prd` viết PRD chuẩn, không phải đoán.

> Đây **chưa phải PRD** — nó là *bộ khung intent* (product definition) mà PRD sẽ dựa vào.

---

## Input (Đầu vào)

- **Ý tưởng bằng ngôn ngữ tự nhiên** — "Tôi muốn người dùng đăng nhập bằng số điện thoại".
- (Tuỳ chọn) **Visual**: link Figma, ảnh wireframe — giúp AI hỏi đúng trọng tâm.
- Context nền đã có: `CLAUDE.md`, `domain-knowledge/`, `project-context.yaml` (nạp tự động qua Gate).

---

## Output (Đầu ra)

| Artifact | Nội dung |
|----------|----------|
| `product-definition/{slug}.md` | Khung intent 8 chặng đã được PO chốt: vấn đề · user · flow · UC · business rule · AC · edge case · scope |

Output này là **input trực tiếp** của `/generate-prd`.

---

## Ai làm gì (Roles & responsibilities)

| Ai | Việc |
|----|------|
| 👤 **PO / BA** | **Lead** — trả lời Q&A, chốt từng chặng, quyết định *cái gì đáng làm* |
| 🤖 **AI** | Co-pilot — hỏi có cấu trúc theo 8 chặng, tổng hợp câu trả lời thành khung intent, phát hiện chỗ mơ hồ để hỏi lại |
| 👤 **SA/Dev (tuỳ chọn)** | Consult — nếu ý tưởng có ràng buộc kỹ thuật lớn |

---

## Câu hỏi cần trả lời (Questions this step answers)

Q&A đi qua **8 chặng**, mỗi chặng chốt xong mới sang chặng sau:

| # | Chặng | Trả lời câu hỏi |
|---|-------|-----------------|
| 1 | **Vấn đề** | Đang giải quyết pain point gì? Tại sao đáng làm? |
| 2 | **User** | Ai dùng? Vai trò/persona nào? |
| 3 | **Flow** | Luồng chính người dùng đi qua là gì? |
| 4 | **Use Case (UC)** | Tính năng gồm những UC nào? |
| 5 | **Business Rule (BR)** | Ràng buộc nghiệp vụ nào chi phối? |
| 6 | **Acceptance Criteria (AC)** | Thế nào là "làm xong đúng"? |
| 7 | **Edge case** | Trường hợp biên/ngoại lệ nào cần lường trước? |
| 8 | **Scope** | Ranh giới: cái gì **trong**, cái gì **ngoài** phạm vi? |

> Toàn bộ nói bằng **ngôn ngữ nghiệp vụ** — chi tiết kỹ thuật (API, retry, timeout) bị **Business Language Guard** đẩy sang bước Design/Tech-Docs.

---

## Framework xử lý thế nào (Mechanics)

1. **Gate chung** (model check, context-loader, checkpoint — xem [README](README.md#khung-chung-mọi-command-common-gate)).
2. **Q&A tuần tự 8 chặng** — AI hỏi từng chặng, chờ PO chốt, **lưu state sau mỗi chặng** để có thể resume nếu ngắt giữa chừng.
3. **Short-circuit** — nếu AI tự thấy một chặng đã đủ rõ, được phép rút gọn nhưng phải nêu lý do.
4. **Tổng hợp** — gộp câu trả lời thành `product-definition/*.md` có cấu trúc.
5. Áp **Business Language Guard** để chặn thuật ngữ kỹ thuật lọt vào.

---

## HITL / Gate

- 🔴 **Mật độ cao nhất toàn pipeline.** Mỗi chặng là một checkpoint — PO xác nhận trước khi đi tiếp.
- Nguyên tắc checkpoint: AI **không hỏi "OK chưa?"** mà hỏi cụ thể — *"UC list đủ chưa? thiếu/thừa UC nào?"*.
- Nếu PO sửa nhiều lần cùng một kiểu → cân nhắc `/learn` để ghi lesson (xem [Feedback Loop](10-feedback-loop.md)).

---

## Ví dụ (Example)

```
PO: /define-product
    "Người dùng quên mật khẩu cần lấy lại qua email"

AI: [Chặng 1 · Vấn đề] Tôi hiểu vấn đề là: user bị khoá ngoài tài khoản khi
    quên mật khẩu, gây mất khách. Đúng không? Có kênh nào ngoài email không?

PO: Đúng. Chỉ email thôi.

AI: [Chặng 4 · UC] Tôi đề xuất 3 UC:
    UC-01 Yêu cầu link reset · UC-02 Đặt lại mật khẩu · UC-03 Hết hạn link.
    Đủ chưa? Có thiếu UC "giới hạn số lần yêu cầu" không?

PO: Thêm UC giới hạn số lần. …
```

Kết quả: `product-definition/quen-mat-khau.md` với 4 UC, BR (link hết hạn 15 phút, tối đa 3 lần/giờ), AC, edge case đã chốt.

---

## Anti-pattern

- ❌ Nhảy thẳng từ ý tưởng sang `/generate-prd` (hoặc tệ hơn: sang code) — mất chặng tư duy user/rule/edge-case.
- ❌ Trả lời chặng cho có, "để AI tự hiểu" — AI sẽ suy diễn và bạn trả giá ở downstream.
- ❌ Nhồi chi tiết kỹ thuật vào đây — sai altitude, để dành cho Tech-Docs.

---

## Bước tiếp theo (Next step)

Khung intent đã chốt → viết PRD:

➡️ [Bước 2 · Specification — `/generate-prd` · `/refine-prd` · `/review-context`](02-specification.md)
