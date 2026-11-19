[← /define-product](01-define-product.md) · [Explain Home](README.md) · [Next: /extend-prd →](02b-extend-prd.md)

# 02 · `/generate-prd` — Sinh Product Requirements Document

> **Một câu.** Biến `product-definition` (8 phase Q&A) thành một **PRD chuẩn nghiệp vụ** đúng template, với đánh số UC/BR và traceability AC↔BR↔UC.

---

## Vấn đề giải quyết

`product-definition` là bản ghi buổi discovery — chưa phải tài liệu chính thức. `/generate-prd` chuyển nó thành **PRD** có cấu trúc cố định (Metadata, AC, UC, BR, Wireframe, Change Log), đánh số nhất quán, và **link traceability** — làm hợp đồng nghiệp vụ để phân rã xuống BDD.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Specification (đầu).
- **Tiền đề:** có `product-definition/{TICKET-ID}-{slug}.md` **đã hoàn tất** (`Status: completed`, Completed Phase 7/7) — còn `in-progress` thì lệnh DỪNG, bảo chạy lại `/define-product` để resume.
- **Gate ra:** PRD sinh với `Status: draft` — cần `/refine-prd` + `/review-context` + PO approve.

---

## Input / Output

**Input:** file product-definition + bảng **Chuẩn hoá thuật ngữ** (Phase 0) + business-dictionary + core-entities.

**Output:** `{specs_dir}/{domain}/{prd-slug}/{TICKET-ID}-{prd-slug}.md` — **một** file PRD ở gốc feature folder (KHÔNG đặt tên `prd.md`).

---

## Các bước xử lý (chi tiết)

Sau Gate + Context-Loader + Business Language Guard:

0. **Phân giải trước khi sinh** — `slug` kế thừa nguyên văn từ tên file product-definition (strip đúng chuỗi `{TICKET-ID}-`, không tách theo dấu `-` đầu); `PO` lấy từ Metadata (trống mới hỏi); **luôn hỏi** link tracker thật (không tự dựng URL Jira từ TICKET-ID); hỏi **một** câu `API Source`: `existing` (xin con trỏ nguồn, AI trích as-is vào Appendix — không truy cập được thì ghi `⛔ PENDING`) · `greenfield` · `partner` song song (đi luồng greenfield + ghi phụ thuộc partner vào §1c, không set `existing` để `/review-tech-docs` không bỏ qua T7). Mục `Xung đột`/`Mục còn thiếu` ở Phase 7 → chuyển thành câu hỏi `[AI DRAFT]` ở "Giả định AI". Rồi CHECKPOINT tóm tắt (ticket, PO, API Source, link tracker) chờ `Y/N`.
1. **Áp Terminology Map từ product-definition** — mỗi cặp `thuật ngữ PO → chuẩn` dùng bản chuẩn khi viết PRD (ưu tiên kể cả khi dictionary vắng).
2. **Thay banned term + chỉ dùng canonical term.**
3. **NEW TERM DETECTION (lưới an toàn)** — nếu vẫn còn thuật ngữ lặp ≥2 lần không có trong dictionary → **DỪNG hỏi PO** (nghĩa là gì, English canonical, bổ sung dictionary?).
4. **Quy tắc Cross-Reference** — mọi TICKET-ID khác được nhắc → phải là inline link tới folder anh em `../{prd-slug-khác}/…`, không để plain text.
5. **Đánh số UC/BR:**
   - UC: `{TICKET}-UC{n}` (n từ 1).
   - BR: `{TICKET}-UC{n}-BR{m}` — **m tăng liên tục toàn PRD, KHÔNG reset theo UC**.
6. **Traceability AC↔BR↔UC (2 chiều bắt buộc):**
   - Mỗi AC remap ref BR từ discovery → BR ID của PRD; mỗi AC có ≥1 ref BR.
   - Mỗi UC liệt kê "AC liên quan"; hai chiều phải **khớp đúng** (lệch = lỗi traceability, sửa trước khi ghi).
7. **Platform Strategy** — PRD thuần WHAT nghiệp vụ; cho phép Wireframe mức nghiệp vụ; chi tiết visual → Design Spec, contract kỹ thuật → Tech Docs.
   - **Ngoại lệ brownfield:** `API Source: existing` → Appendix "Existing API Contract" được chứa chi tiết kỹ thuật (trích as-is).
8. **Generate** — ghi PRD theo template: Metadata (Version 1.0, Status draft, Domain, Ticket, API Source) · Feature · §1 Tổng quan (User Story, Scope, Phụ thuộc liên service) · §2 AC · §3 UC (BR table 3 cột: ID | Business Rule | Business Logic) · Wireframe · Change Log. Kiểm Quality Checklist trước khi ghi.

---

## Checkpoint & Gate

- Nhãn khai **chặn CỨNG** — chỉ áp khi PRD **đã tồn tại**, và khi đó lệnh **DỪNG hẳn, không hỏi Y/N** (xem cuối trang). PRD chưa có → CHECKPOINT tóm tắt trước khi sinh (`Y/N`).
- DỪNG khi: product-definition chưa `completed` · NEW TERM · mở cột bảng BR trên PRD đã có BDD (hỏi chọn).
- Output `Status: draft` — **chưa** mở khoá downstream.

---

## Cơ chế đặc biệt

- **BR đánh số liên tục toàn PRD** (không reset) — để BR ID tự mang thông tin UC, suy ngược traceability.
- **AC↔BR↔UC nhất quán 2 chiều** — self-check ngay khi sinh.
- **Business Rule = bảng 3 cột** (không tách Business Logic ra khối riêng) — giữ altitude. 3 cột là **mặc định**: khi mọi BR trong một UC chia cùng một bộ thuộc tính lặp (dấu hiệu: phải nhồi nhiều bản ghi bằng `<br/>` vào một ô) thì **mở cột**, một bản ghi một dòng; giá trị dùng chung đưa lên **§1d Quy ước**. Mở cột trên PRD đã có BDD → DỪNG hỏi, vì số BR tăng sẽ đánh số lại BR phía sau.
- **Brownfield exception** — chỉ Appendix "Existing API Contract" được chứa kỹ thuật.

---

## 👓 Góc nhìn tối ưu

- **NEW TERM detection lặp lại** ở cả `/define-product` (Phase 0/3) lẫn đây (lưới an toàn). Nếu discovery làm tốt, bước này hiếm khi kích hoạt — nhưng vẫn tốn prompt. Cân nhắc: đây là redundancy có chủ đích (an toàn) hay dư thừa?
- **Traceability 2 chiều tự kiểm** là điểm mạnh — có thể là mẫu để nhân sang các artifact khác.
- **Phụ thuộc chất lượng product-definition** — nếu discovery sơ sài, PRD sẽ mỏng; `/generate-prd` không tự bù bằng Q&A (đó là việc `/refine-prd`).
- **Business Language Guard chạy lại** ở đây dù đã chạy ở discovery — chi phí lặp.

---

## Kết nối

**Trước:** [`/define-product`](01-define-product.md) · **Sau:** [`/refine-prd`](03-refine-prd.md) → [`/review-context`](04-review-context.md).

> **PRD đã tồn tại?** Lệnh này **từ chối chạy** — dùng [`/extend-prd`](02b-extend-prd.md). Ghi đè sẽ mất `# Change Log` + rollover, Version/Status thật, và **đánh số lại BR từ đầu** (phá mọi `@trace.business_rules` trong `bdd/` đã sinh). Ba mất mát đều không hoàn tác được từ trong lệnh → dừng hẳn, **không hỏi Y/N**.
