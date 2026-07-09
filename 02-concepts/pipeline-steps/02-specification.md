[← Discovery](01-discovery.md) · [Pipeline Steps](README.md) · [Next: Design-Spec →](03-design-spec.md)

# Bước 2 · Specification — Hình thành đặc tả (PRD)

> **Tóm tắt.** Biến khung intent thành **PRD** chuẩn nghiệp vụ, tinh chỉnh qua 3 lăng kính, rồi qua **gate chất lượng** để PO đóng dấu `approved`.
> **Commands:** `/generate-prd` → `/refine-prd` → `/review-context`

| | |
|---|---|
| **Giai đoạn** | Specification |
| **Owner** | 👤 Product Owner (SA/Dev review) |
| **Đầu vào** | `product-definition/*.md` (từ Discovery) |
| **Đầu ra** | PRD `Status: draft → approved` |
| **HITL** | 🔴 Cao — `/review-context` sạch critical + PO approve |

---

## Mục đích (Purpose)

PRD là **hợp đồng nghiệp vụ** giữa PO ↔ Dev ↔ AI. Đây là **cổng chất lượng cuối cùng ở thượng nguồn** trước khi phân rã xuống BDD/code. Nếu PRD mơ hồ/lệch, mọi code sinh ra sau đó là "rác hàng loạt". Bước này:

- Viết PRD **thuần ngôn ngữ nghiệp vụ**, đúng cấu trúc, có traceability.
- Bắt lỗi **trước** khi truyền xuống — qua 3 lăng kính DEV/SA/PO và findings có mã.
- Chốt một trạng thái `approved` rõ ràng để mở khoá downstream.

---

## Ba lệnh, ba vai trò (Three commands)

| Lệnh | Vai trò | Kết quả |
|------|---------|---------|
| `/generate-prd` | **Sinh** PRD draft từ product-definition | PRD `Status: draft` |
| `/refine-prd` | **Tinh chỉnh** qua 3 lăng kính DEV/SA/PO (fan-out per-UC) | Findings để PO accept/reject |
| `/review-context` | **Gate chất lượng** — findings P0–P5, phải sạch critical | PO đặt `Status: approved` |

---

## Input (Đầu vào)

- `product-definition/{slug}.md` từ [Discovery](01-discovery.md).
- Business dictionary / domain-knowledge (nạp tự động).
- (Umbrella) metadata routing: `Domain` khớp một service key.

## Output (Đầu ra)

| Artifact | Nội dung |
|----------|----------|
| `specs/{domain}/{prd-slug}/{TICKET-ID}-{prd-slug}.md` | PRD: **Metadata · AC · UC · BR · Wireframe · Change Log** |
| `.agent/review/*-findings.yaml` | Findings của `/refine-prd` & `/review-context` (để Review Board) |
| Trạng thái `| Status | approved |` trong Metadata | 🔒 Gate mở khoá `/generate-bdd` |

---

## Ai làm gì (Roles & responsibilities)

| Ai | Việc |
|----|------|
| 👤 **PO** | Lead: duyệt scope/terminology, accept/reject findings, đặt `Status: approved` |
| 👤 **SA** | Lăng kính SA trong `/refine-prd` — đọc PRD bằng mắt kiến trúc, viết lại bằng lời nghiệp vụ |
| 👤 **Dev** | Lăng kính DEV — bắt chỗ mơ hồ sẽ gây khó khi hiện thực |
| 🤖 **AI** | Draft PRD, fan-out mỗi UC/lăng kính một sub-agent, completeness-critic, sinh findings |

---

## Câu hỏi cần trả lời (Questions this step answers)

- Scope/terminology đã đúng và nhất quán với business dictionary chưa?
- Có UC/BR/AC nào mơ hồ, mâu thuẫn, hay lạc tầng (altitude) không?
- PRD đã đủ cấu trúc để phân rã xuống BDD chưa?
- (Umbrella) PRD route về đúng service chưa?

---

## Framework xử lý thế nào (Mechanics)

**`/generate-prd`** — Round Q&A + confirm domain/terminology → sinh PRD draft đúng template, áp Business Language Guard.

**`/refine-prd`** — fan-out **mỗi UC × 3 lăng kính**:
- **DEV** — chỗ nào khó hiện thực / thiếu định nghĩa?
- **SA** — rủi ro kiến trúc, phụ thuộc, nhất quán hệ thống (nhưng viết bằng lời nghiệp vụ).
- **PO** — giá trị, scope, hoàn chỉnh nghiệp vụ.
- Rồi một vòng **completeness-critic**: "đã đủ chưa / có lộn tầng không". PO accept/reject từng finding qua **Review Board** (findings YAML), có thể **resume** để áp fix.

**`/review-context` (PRD)** — findings có mã, sạch **critical** mới cho qua:

| Mã | Check |
|----|-------|
| **P0** | Umbrella routing (chỉ mode umbrella) — chạy trước, là gate check |
| **P1** | Terminology (Business Dictionary) |
| **P2** | Ambiguity (mơ hồ) |
| **P3** | Domain conflict (xung đột cross-PRD) |
| **P4** | Structural completeness (đủ section/metadata) |
| **P5** | Custom criteria (tuỳ chọn) |

> P0 & P3 do orchestrator chạy (cần config/PRD khác); P1/P2/P4/P5 fan-out mỗi nhóm một sub-agent.

---

## HITL / Gate

- 🛑 `/refine-prd`: PO accept/reject **từng** finding — không auto-apply.
- 🔒 `/review-context`: phải **sạch critical**, sau đó **PO tự đặt** `| Status | approved |`. Không có lệnh `/approve-prd` — "duyệt" = người đặt field trạng thái.
- Downstream (`/generate-bdd`) đọc field này; `draft` → cảnh báo mềm.

---

## Ví dụ (Example)

```
/generate-prd quen-mat-khau        → PRD draft (4 UC, BR, AC)
/refine-prd                        → 12 findings (3 lăng kính)
   → PO accept 9, reject 3 → /refine-prd (resume) áp 9 fix
/review-context <PRD>              → P2: "AC UC-02 mơ hồ 'nhanh chóng'"
   → PO sửa → chạy lại → sạch critical
   → PO đặt Status: approved  🔒 mở khoá BDD
```

---

## Anti-pattern

- ❌ Để chi tiết kỹ thuật (retry/timeout/API) trong PRD — lạc altitude; đẩy xuống Tech-Docs.
- ❌ Đặt `approved` khi còn finding critical — phá gate, hỏng downstream.
- ❌ Bỏ `/refine-prd` "vì PRD trông ổn" — 3 lăng kính bắt lỗi mắt thường bỏ sót.

---

## Bước tiếp theo (Next step)

PRD `approved` → nếu là FE/App: [Design-Spec](03-design-spec.md); nếu không: thẳng tới [BDD](04-bdd.md).
