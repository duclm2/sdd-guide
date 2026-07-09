[← BDD](04-bdd.md) · [Pipeline Steps](README.md) · [Next: Code →](06-code.md)

# Bước 5 · Tech-Docs — Thiết kế kỹ thuật (Technical Design)

> **Tóm tắt.** Từ BDD `approved`, sinh **một tech-design full-stack gộp cho cả PRD** — API contract, entity, data, dependency — rồi review đa chiều + **cổng ký liên team** cho contract cross-service.
> **Commands:** `/generate-tech-docs` → `/review-tech-docs`

| | |
|---|---|
| **Giai đoạn** | Design (đầu ra kỹ thuật) |
| **Owner** | 👤 SA / Tech Lead |
| **Đầu vào** | BDD `approved` + entity catalog + CLAUDE.md |
| **Đầu ra** | `tech-docs/{TICKET-ID}-tech-design.md` (một doc full-stack/PRD) |
| **HITL** | 🟠 Vừa — review đa chiều + cổng ký T7 cho contract liên team |

---

## Mục đích (Purpose)

Đây là nơi **chuyển từ ngôn ngữ nghiệp vụ sang ngôn ngữ kỹ thuật** — ranh giới cuối cùng của Business Language Guard. Tech-design là **API contract** giữa các team: BE viết, FE/App đọc. Chốt sai contract ở đây = đốt budget code cho cả hai phía. Bước này:

- Sinh **một** blueprint full-stack gộp cho cả PRD (BE API/data/DB **và** client), phủ mọi UC.
- Chuẩn hoá **entity catalog, relationship map, enum registry** làm nguồn chân lý cho code.
- **Reverse-document** khi API đã tồn tại (mô tả as-is thay vì thiết kế mới).
- Với dependency cross-service → có thể sinh **System BDD**.

---

## Input (Đầu vào)

- **BDD `approved`** của mọi platform trong PRD.
- **Entity catalog** + relationship map + enum registry (domain-knowledge).
- `CLAUDE.md` (§2 kiến trúc) để bám layer/convention của stack.
- (Nếu reverse-document) mã nguồn API hiện có.

## Output (Đầu ra)

| Artifact | Nội dung |
|----------|----------|
| `specs/{domain}/{prd-slug}/tech-docs/{TICKET-ID}-tech-design.md` | **Một doc full-stack** phủ mọi UC: API endpoint, DTO, data model, DB, dependency, §10 UC Coverage |
| `@trace.status: approved` | 🔒 Mở khoá `/generate-code` |
| (Tuỳ chọn) System BDD | Cho dependency cross-service |

> Contract này là **artifact liên team**: BE viết → FE/App đọc ở `/generate-code --phase=integration`. Trong umbrella, nó nằm ở **spec repo dùng chung**.

---

## Ai làm gì (Roles & responsibilities)

| Ai | Việc |
|----|------|
| 👤 **SA / Lead** | Lead: quyết trade-off thiết kế, duyệt contract, **ký** cổng liên team (T7) |
| 👤 **Dev các team** | Review contract phần mình tiêu thụ/cung cấp |
| 🤖 **AI** | Sinh tech-design từ BDD + entity catalog; reverse-document API as-is; review đa chiều |

---

## Câu hỏi cần trả lời (Questions this step answers)

- Mỗi UC hiện thực bằng **API/endpoint/DTO** nào?
- **Entity/field/quan hệ/enum** chuẩn là gì (nguồn chân lý cho code)?
- Contract giữa BE ↔ FE/App có **khớp** và **đủ** cho mọi scenario không?
- Dependency **cross-service** nào cần System BDD + ký liên team?

---

## Framework xử lý thế nào (Mechanics)

**`/generate-tech-docs`**
1. Đọc mọi `.feature approved` của PRD + entity catalog.
2. Sinh **một** tech-design gộp full-stack, mục **§10 UC Coverage** ánh xạ finding/section về từng UC.
3. Nếu API đã tồn tại → **reverse-document** (mô tả as-is, không tự chế shape).
4. Chuẩn hoá entity/DTO/endpoint theo catalog để nhất quán với PRD/BDD.

**`/review-tech-docs`** — review **đa chiều**, findings gom theo từng UC (đọc §10 UC Coverage):
- Kiểm tính đủ/đúng của contract, entity, error, dependency.
- **T7 — cổng ký liên team**: contract cross-service phải được các team liên quan **ký** trước khi code.

---

## HITL / Gate

- 🟠 SA duyệt tech-design; **read-only** review — chỉ báo findings, không tự sửa.
- 🔒 **T7 sign-off**: contract liên team chưa ký → không mở khoá code phía tiêu thụ.
- `@trace.status: approved` trên tech-design → `/generate-code` dùng §4 làm nguồn contract; `draft/in-review` hoặc còn blocker-GAP → chỉ WARN (không chặn).

---

## Anti-pattern

- ❌ Tự "chế" shape DTO/endpoint khi API đã tồn tại — phải reverse-document as-is.
- ❌ Bỏ cổng ký T7 rồi để hai team hiểu contract khác nhau → rework tốn kém.
- ❌ Sinh code khi tech-design còn `draft` với contract chưa chốt.

---

## Bước tiếp theo (Next step)

Tech-design `approved` (+ ký T7) → sinh code:

➡️ [Bước 6 · Code — `/generate-code`](06-code.md)
