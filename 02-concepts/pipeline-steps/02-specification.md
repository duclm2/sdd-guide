[← Discovery](01-discovery.md) · [Pipeline Steps](README.md) · [Next: Design-Spec →](03-design-spec.md)

# Bước 2 · Specification — Hình thành đặc tả (PRD)

> **Tóm tắt.** Biến khung intent thành **PRD** chuẩn nghiệp vụ, tinh chỉnh qua 3 lăng kính, rồi qua **gate chất lượng** để PO đóng dấu `approved`.
> **Commands:** `/generate-prd` (lần đầu) · `/extend-prd` (**thêm** vào PRD đã có) · `/amend-prd` (**đổi** yêu cầu đã có) → `/refine-prd` → `/review-context`

> **Chọn lệnh nào — bốn nhánh, phân biệt bằng THAO TÁC GHI:**
>
> | Tình huống | Lệnh | Ghi kiểu gì | Vì sao không dùng cái kia |
> |---|---|---|---|
> | PRD **chưa tồn tại** | `/generate-prd` | **Write** cả file | — |
> | PRD đã có, **THÊM** UC/AC/BR mới | **`/extend-prd`** | Edit **add-only** — output là **superset chặt** | `/generate-prd` **từ chối chạy** trên file đã có |
> | PRD đã có, **ĐỔI** một yêu cầu đang đúng cú pháp | **`/amend-prd`** | Edit **tại chỗ** — output **KHÔNG** phải superset | `/extend-prd` chỉ add-only; `/refine-prd` chỉ áp finding của chính nó |
> | PRD đã có, sửa **vấn đề review đã soi ra** | `/refine-prd` → Review Board → `--resume` | Edit trong phạm vi finding | `/refine-prd` **không thêm/đổi được** theo ý định mới — nó tự cấm đụng section ngoài findings |
>
> **Xoá hẳn một BR/AC → không có lệnh, và có chủ ý:** xoá row làm `@trace.business_rules` trong `.feature` trỏ vào ID không còn ⇒ `TRACE_ORPHAN` 🔴. Dùng `/amend-prd --retire {ID}` — khai tử **tại chỗ**, giữ nguyên row + ID.
>
> ⚠️ **Sửa tay file `.md` là điểm mù (GAPS-v4 G54).** Mọi drift detector so **nhãn version**, không so nội dung (0 content hash trong codebase) — sửa mà không bump version ⇒ **0 cờ**. `/validate-traces` Step 3.9 canh cửa sau bằng cờ 🔴 `PRD_UNTRACKED_EDIT`.
>
> **`/generate-prd` dừng hẳn (không hỏi Y/N) nếu file đã tồn tại.** Ghi đè sẽ mất `# Change Log` + rollover, Version/Status thật, và **đánh số lại BR từ đầu** — cái cuối lan **ra ngoài file**, phá mọi `@trace.business_rules` trong `bdd/` đã sinh. Ba mất mát đều không hoàn tác được từ trong lệnh, nên không đặt sau một phím bấm.

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
| `/generate-prd` | **Sinh** PRD draft từ product-definition. Từ chối chạy nếu PRD đã tồn tại | PRD `Status: draft` |
| `/extend-prd` | **Thêm** UC/AC/BR vào PRD đã duyệt — đánh số **nối tiếp**, ghi **add-only** + guard sau-ghi, drain `feedback/prd-change-requests/` | PRD v+1, `Status → draft` |
| `/amend-prd` | **Đổi tại chỗ** một AC/BR/UC đã duyệt — PO khai tường minh `amend_targets`, kiểm va chạm, guard sau-ghi **HAI CHIỀU** (Bảo toàn + Giới hạn). `--retire {ID}` khai tử tại chỗ | PRD v+1, `Status → draft` |
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

**`/generate-prd`** — Round Q&A + confirm domain/terminology → sinh PRD draft đúng template, áp Business Language Guard. **Guard đầu vào:** file đã tồn tại → dừng hẳn, chỉ sang `/extend-prd`.

**`/extend-prd`** — 7 bước, tái dùng tối đa thứ đã có:

1. Nạp trạng thái: `max_uc` / `max_br` / `max_ac`, UC hiện có, changelog, BDD đã sinh cho những UC nào
2. **Drain hàng đợi** `prd-change-requests/` — `accepted` → nguyên liệu; `Open` → trình PO kèm số ngày chờ
3. Discovery delta — tái dùng Phase 1/4/5/6 của `/define-product` (bỏ Phase 0/2/7 vốn là toàn-feature), **cộng phase "kiểm va chạm"**
4. **Đánh số nối tiếp** — ràng buộc cứng nhất
5. Ghi **Edit add-only** + guard sau-ghi
6. Bump version + changelog — tái dùng **nguyên** `/refine-prd` Phase 3
7. Report: nêu rõ **UC không đổi** + route `--realign-prd-version` cho chúng

**Phase "kiểm va chạm" — `/define-product` không có.** Lúc discovery lần đầu chưa có gì để va chạm; ở đây thì có. Ba câu bắt buộc:

| Câu | Vì sao |
|---|---|
| Phần thêm làm một BR cũ **sai đi** không? | *(BR cũ "tối đa 5", phần mới cần 20)* → đây là **sửa** BR cũ, không phải thêm mới → bump **major** |
| Đã có UC/AC nào **phủ một phần** chưa? | → hỏi PO: mở rộng UC cũ hay tạo mới. **Không tự quyết** |
| Cần dữ liệu/năng lực từ đâu khác? | → ghi vào §1c Phụ thuộc liên service |

Va chạm âm thầm là cách một PRD **tự mâu thuẫn với chính nó**.

**Hai ràng buộc cứng:**

- **Không đánh lại BẤT KỲ ID cũ nào.** UC/AC/BR mới đều là `max + 1`. Số bị bỏ trống (do UC bị xoá ở version trước) **để trống vĩnh viễn** — ID đã từng tồn tại có thể còn bị tham chiếu ở BDD, code, bug report, hoặc PRD khác.
- **Dòng changelog phải nêu rõ UC/AC/BR.** Đây là **contract**: `/generate-bdd` đọc để quyết cập nhật hẹp (Y) hay gen lại toàn bộ (F), và `/validate-traces` Step 4/5 đọc để lọc `PRD_DRIFT` 🟠 vs `PRD_STALE_REF` ⓘ. Một dòng mơ hồ (`"cập nhật theo yêu cầu mới"`) làm **mất cả hai** bộ lọc cùng lúc.

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
