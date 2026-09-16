[← Design-Spec](03-design-spec.md) · [Pipeline Steps](README.md) · [Next: Tech-Docs →](05-tech-docs.md)

# Bước 4 · BDD — Kịch bản hành vi (Behavior Specification)

> **Tóm tắt.** Phân rã PRD thành các file `.feature` (Gherkin) mang trace metadata — đây là **anchor cứng** mà mọi code/test sau này link về.
> **Commands:** `/generate-bdd` → `/review-context`

| | |
|---|---|
| **Giai đoạn** | Design |
| **Owner** | 👤 PO (+ Dev review) |
| **Đầu vào** | PRD `approved` (+ design-spec nếu FE/App) |
| **Đầu ra** | `bdd/**/*.feature` với `@trace.*`; `@trace.status: approved` |
| **HITL** | 🔴 Cao — 🛑 UC decomposition + gate findings B1–B6 |

---

## Mục đích (Purpose)

BDD là **trái tim của traceability**. Mỗi scenario là một đơn vị hành vi có thể kiểm chứng; mỗi dòng code sau này link `@trace.source` về một scenario. Cấu trúc UC/SC quyết ở đây **tác động tới mọi file code phía sau** — nên đây là checkpoint nặng.

- Phân rã PRD → **UC → Scenario** bằng Gherkin (Given/When/Then).
- Gắn **trace metadata** (`@trace.id/scenario/business_rules/bdd_version/prd_version`).
- Gate B1–B6 đảm bảo BDD phủ đủ PRD, đúng thuật ngữ, đúng luật Gherkin.

---

## Input (Đầu vào)

- **PRD `approved`** (gate mềm — cảnh báo nếu `draft`, áp mọi mode).
- **Design-spec** (nếu FE/App) — lái coverage UI.
- `feature.template` (SoT skeleton) + `bdd-writing-guide.md`.

## Output (Đầu ra)

| Artifact | Nội dung |
|----------|----------|
| `specs/{domain}/{prd-slug}/bdd/{web\|app\|system}/{UC-ID}*.feature` | Scenario Gherkin + tag `@trace.*` + Coverage Matrix |
| `.trace/{domain}/{prd-slug}/{UC-ID}-{platform}.tsv` | Sổ trace — **một sổ cho mỗi UC × platform** |
| `@trace.status: approved` (sau review) | 🔒 Mở khoá `/generate-tech-docs` |

> **Subfolder `{platform}/` LUÔN có — mọi mode, kể cả umbrella.** `web` và `system` của cùng một UC là **hai file khác nhau**; bỏ subfolder thì chúng ra cùng filename và **ghi đè nhau**. Trace cũng tách theo platform, nên bố cục spec phải khớp. Mỗi `.feature` còn mang `@trace.platform` **khớp** segment path của chính nó.
>
> Ở umbrella mode, `active_platform` được **suy từ `active_module`** (react/vue/… → `web` · flutter/RN/… → `app` · java-spring/golang/… → `system`); không suy được thì lệnh **dừng và hỏi**, không ghi file.
>
> *Project còn ở bố cục phẳng cũ (`bdd/*.feature`): chạy `npx @educa-corp/sdd-framework --migrate-bdd-platform` — dry-run mặc định.*

---

## Ai làm gì (Roles & responsibilities)

| Ai | Việc |
|----|------|
| 👤 **PO** | Quyết cấu trúc UC/SC; xác nhận decomposition đúng intent |
| 👤 **Dev** | Lăng kính review BDD (side-effect, khả thi) |
| 🤖 **AI** | Phân rã UC→SC; PRD lớn → orchestrator **spawn mỗi UC một sub-agent**; sinh Gherkin + trace |

---

## Câu hỏi cần trả lời (Questions this step answers)

- PRD phân rã thành những **UC/Scenario** nào? Có thiếu/thừa không?
- Mỗi scenario có phủ một **AC/BR** cụ thể không (B1 coverage)?
- Gherkin có đúng luật (R1–R10), đúng thuật ngữ (business dictionary) không?
- Có **side-effect** nào chưa được scenario mô tả không (B6)?

---

## Framework xử lý thế nào (Mechanics)

**`/generate-bdd`**
1. Gate: kiểm PRD `Status: approved` (cảnh báo mềm mọi mode).
2. 🛑 **UC decomposition checkpoint** — trình danh sách UC/SC để PO chốt.
3. PRD lớn → **fan-out**: orchestrator giao mỗi UC cho một sub-agent (`_agent_mode`) sinh song song.
4. Sinh `.feature` theo `feature.template`, gắn `@trace.*`, dựng Coverage Matrix.
5. **Gen lại:** bump `@trace.sc_version` **+0.1 cho từng SC có thân đổi** (xem dưới); SC bị xoá mà **đã có code** → giữ row `.tsv`, đặt `status = ORPHANED` (không xoá row — xoá đi thì code thành vô hình).
6. Có thể incorporate **scenario proposal** đã `accepted` từ tester — khi chèn phải **normalize**: gán `sc_id` kế tiếp, strip `@proposed`/`@from-test`, append row `.tsv` (xem [Feedback Loop](10-feedback-loop.md)).

### `sc_version` — tín hiệu DRIFT duy nhất

Mỗi scenario mang `@trace.sc_version` riêng. Nó là **thứ duy nhất** cho `/validate-traces` biết code của SC đó đã lỗi thời (`spec_ver != gen_ver` → `DRIFT`).

| Đổi cái gì | Bump? |
|---|:---:|
| Tên `Scenario:` · chuỗi step · data table · `# Side-effects:` | ✅ **+0.1** |
| `@trace.business_rules` · tag `@happy`/`@edge` · comment | ❌ không |
| Header file · Coverage Matrix · gom NHÓM | ❌ không (đó là `bdd_version`) |

**Ai bump:** `/generate-bdd` (khi gen lại, so 4 thành phần trên với bản trên disk) · `/review-context --fix`/`--resume` (mỗi SC có finding đổi thân) · **người sửa tay** (có mục trong Pre-merge Checklist).

> Sửa SC mà quên bump → code sinh từ bản cũ **vĩnh viễn hiện `OK`**, không ai biết phải regen. Bump vô cớ thì ngược lại: mọi SC hiện `DRIFT` giả và cờ mất giá trị.
>
> Phân biệt với `@trace.bdd_version` (**cấp file**): nó bắt thay đổi mà `sc_version` không thấy — Background, `@trace.dataset`, Business Definition. Cả hai đều cần.

**`/review-context` (BDD)** — findings có mã, sạch critical mới `approved`:

| Mã | Check |
|----|-------|
| **B1** | PRD coverage — mọi AC/UC được phủ |
| **B2** | Terminology & entity (đúng dictionary) |
| **B3** | Gherkin rules (R1–R10) |
| **B4** | Compliance (C.1–C.5) |
| **B5** | Metadata & structural (đủ `@trace` header) |
| **B6** | Side-effect completeness |

> B1–B6 fan-out mỗi nhóm một sub-agent quét toàn file cho nhóm đó.

---

## HITL / Gate

- 🛑 **UC decomposition** — điểm dừng quan trọng nhất: cấu trúc sai ở đây nhân xuống mọi code.
- 🔒 `/review-context` sạch **critical** → đặt `@trace.status: approved` → mở khoá Tech-Docs.

---

## Ví dụ (Example)

```gherkin
@trace.id=UC-02 @trace.scenario=SC-02.1
@trace.business_rules=BR-03 @trace.bdd_version=1.0 @trace.prd_version=1.2
Scenario: Đặt lại mật khẩu với link còn hạn
  Given người dùng mở link reset còn hiệu lực
  When nhập mật khẩu mới hợp lệ và xác nhận
  Then hệ thống cập nhật mật khẩu và cho đăng nhập lại
```

---

## Anti-pattern

- ❌ Sinh code khi BDD còn `draft` (chưa `approved`).
- ❌ Nhét thuật ngữ kỹ thuật (selector, API, retry) vào step Gherkin — vi phạm B3/R3.
- ❌ Bỏ qua UC decomposition checkpoint — cấu trúc UC lệch kéo cả pipeline lệch.

---

## Bước tiếp theo (Next step)

BDD `approved` → thiết kế kỹ thuật:

➡️ [Bước 5 · Tech-Docs — `/generate-tech-docs` · `/map-testids` · `/review-tech-docs`](05-tech-docs.md)
