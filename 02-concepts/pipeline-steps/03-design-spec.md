[← Specification](02-specification.md) · [Pipeline Steps](README.md) · [Next: BDD →](04-bdd.md)

# Bước 3 · Design-Spec — Đặc tả thiết kế UI (chỉ FE/App)

> **Tóm tắt.** Chốt **chi tiết visual bám Figma thật** trước khi sinh BDD giao diện — chỉ áp dụng cho service **Frontend / App**.
> **Command:** `/generate-design-spec`

| | |
|---|---|
| **Giai đoạn** | Design (nhánh FE/App) |
| **Owner** | 👤 PO / PM (+ Designer) |
| **Đầu vào** | PRD `approved` + Figma/wireframe |
| **Đầu ra** | `design-spec/` — spec visual 2 tầng ngôn ngữ |
| **HITL** | 🟠 Vừa — checkpoint mềm nếu design-spec chưa `approved`/lỗi thời |
| **Bỏ qua khi** | Service là **backend/system** (không có UI) |

---

## Mục đích (Purpose)

BDD giao diện (`bdd/web/`, `bdd/app/`) cần biết **màn hình có gì, trạng thái nào, tương tác ra sao**. Nếu để `/generate-bdd` tự đoán UI, coverage sẽ thiếu hoặc lệch Figma. Design-Spec:

- Ghi lại **chi tiết visual thật** (component, state, empty/error/loading, navigation) bám Figma.
- Dùng **2 tầng ngôn ngữ**: mô tả nghiệp vụ cho PO + tham chiếu component cho Dev/Designer.
- **Lái coverage UI** của BDD — mỗi trạng thái màn hình → scenario tương ứng.

---

## Input (Đầu vào)

- **PRD `approved`** (bước 2).
- **Figma/wireframe thật** — nguồn chân lý về visual.
- `domain-knowledge/figma-components/{stack}.md` — quy ước map component theo stack (react, flutter, vue…).

## Output (Đầu ra)

| Artifact | Nội dung |
|----------|----------|
| `specs/{domain}/{prd-slug}/design-spec/{TICKET-ID}-design-spec-{platform}-{slug}.md` | Đặc tả từng màn hình: layout, component, state, interaction, navigation |
| Trạng thái `approved` | 🔒 Mềm — `/generate-bdd` cảnh báo nếu design-spec chưa duyệt/lỗi thời |

---

## Ai làm gì (Roles & responsibilities)

| Ai | Việc |
|----|------|
| 👤 **PO / PM** | Duyệt design-spec bám đúng Figma & intent |
| 👤 **Designer** | Nguồn Figma; xác nhận component/state |
| 🤖 **AI** | Đọc Figma + PRD → sinh design-spec 2 tầng, map component theo stack |

---

## Câu hỏi cần trả lời (Questions this step answers)

- Màn hình gồm những **component & trạng thái** nào (default/empty/loading/error)?
- Luồng **navigation** giữa các màn ra sao?
- Mỗi trạng thái ứng với **scenario UI** nào mà BDD phải phủ?
- Visual có **khớp Figma** không?

---

## Framework xử lý thế nào (Mechanics)

1. Gate chung + xác định service FE/App.
2. Đọc PRD `approved` + Figma + `figma-components/{stack}.md`.
3. Sinh design-spec **2 tầng ngôn ngữ**: mô tả nghiệp vụ (PO đọc) + tham chiếu component (Dev dùng).
4. Đánh dấu các **trạng thái màn hình** để `/generate-bdd` phủ đủ.

---

## HITL / Gate

- 🟠 Checkpoint **mềm**, hai chỗ: `/generate-design-spec` cảnh báo nếu **PRD** chưa `approved` (vẫn cho prototype song song); `/generate-bdd` FE/App cảnh báo nếu design-spec **chưa `approved`**, còn màn ❌ Missing Figma, hoặc **lỗi thời** so với PRD (`Built from PRD` ≠ Version PRD hiện tại). PO xác nhận trước khi tiếp.
- Design-spec chỉ được đặt `approved` khi PO + Designer sign-off và **0 màn ❌ Missing**. Sửa design-spec sau khi BDD/code đã sinh → `/validate-traces` Step 5d gắn cờ `DESIGNSPEC_DRIFT` / `DESIGNSPEC_STALE_VS_BDD`.

---

## Anti-pattern

- ❌ Sinh design-spec khi Figma chưa chốt — spec lệch, phải làm lại.
- ❌ Chạy bước này cho service backend — không có UI để đặc tả.
- ❌ Nhét business rule mới vào design-spec — rule thuộc PRD, đây chỉ visual.

---

## Bước tiếp theo (Next step)

Design-spec chốt → sinh BDD giao diện:

➡️ [Bước 4 · BDD — `/generate-bdd`](04-bdd.md)
