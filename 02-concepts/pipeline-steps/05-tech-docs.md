[← BDD](04-bdd.md) · [Pipeline Steps](README.md) · [Next: Code →](06-code.md)

# Bước 5 · Tech-Docs — Thiết kế kỹ thuật (Technical Design)

> **Tóm tắt.** Từ BDD `approved`, sinh **một tech-design full-stack gộp cho cả PRD** — API contract, entity, data, dependency — rồi review đa chiều + **cổng ký liên team** cho contract cross-service.
> **Commands:** `/generate-tech-docs` → `/map-testids` → `/review-tech-docs`

| | |
|---|---|
| **Giai đoạn** | Design (đầu ra kỹ thuật) |
| **Owner** | 👤 SA / Tech Lead |
| **Đầu vào** | BDD `approved` + entity catalog + CLAUDE.md |
| **Đầu ra** | `tech-docs/{TICKET-ID}-tech-design.md` (một doc full-stack/PRD) + **hợp đồng test-id** (`@trace.testid_attr` ở header · §4.5.6 Test Selectors) |
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
| `@trace.testid_attr` (header) + **§4.5.6 Test Selectors** | **Hợp đồng test-id giữa FE và QC** — do `/map-testids` ghi. Header giữ **TÊN thuộc tính** (một giá trị cho cả doc); §4.5.6 giữ **GIÁ TRỊ** test-id từng element + cột *Serves SC* |
| `@trace.status: approved` | 🔒 Mở khoá `/generate-code` **và** `/qc-design-test` — hai nhánh chạy song song |
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

**`/map-testids`** — chốt **hợp đồng test-id** trước khi có dòng code nào:
1. Ghi `@trace.testid_attr` ở header — **tên** thuộc tính của stack client (`data-testid` · `data-test` · `testID` · `Key`…), **một** giá trị cho cả doc.
2. Ghi **§4.5.6 Test Selectors** — **giá trị** test-id từng element, kèm cột **Serves SC (UC · SC)** làm chỉ mục ngược.
3. Nếu §4.5.6 đã có dòng cũ và giá trị mới **lệch** → **DỪNG, không ghi đè**, in cả hai bản cho người quyết.
4. Ghi lại §4.5.6 làm **mất hiệu lực** kết quả QC cũ: `qc_status` → `not_run`, `qc_run_at` → `—` *(không đụng `qc_owner`/`qc_blocked_by`)*. Test-script bám selector cũ đã không còn đúng — để nguyên `pass` là nói dối.
5. `--from-code` là chế độ **ngược**, dùng cho brownfield: đọc test-id đã có trong code UI rồi ghi ngược vào doc.

> **Vì sao lệnh này nằm ở đây chứ không sau `/generate-code`.** Chốt hợp đồng **trước** code thì FE (gắn attribute) và QC (viết test case + script) đọc **cùng một bản đã đóng băng** và **chạy song song**. Chốt sau code thì QC phải ngồi chờ, rồi tự dò selector từ DOM — script giòn, dev đổi một class là vỡ, và **không ai báo**.

**`/review-tech-docs`** — review **đa chiều**, findings gom theo từng UC (đọc §10 UC Coverage):
- Kiểm tính đủ/đúng của contract, entity, error, dependency.
- **T3 — BDD traceability**: design có khớp **nội dung** scenario không (2 chiều, match trong đúng lane platform).
- **T3b — BDD freshness**: doc này dựng từ BDD **version nào**, BDD giờ ở version nào.
- **T6 — hợp đồng test-id**: §4.5.6 rỗng, hoặc header thiếu `@trace.testid_attr` → **Major**, và **không tự sửa được** (phải chạy `/map-testids`). Đây là cổng giữ cho hợp đồng không bị bỏ trống rồi trôi xuống `/generate-code`.
- **T7 — cổng ký liên team**: contract cross-service phải được các team liên quan **ký** trước khi code.

### T3b — vì sao độ tươi cần một cổng riêng

Header tech-doc mang `@trace.bdd_versions` — **map theo platform** (`system=1.5, web=1.9`), số nhiều, cố ý khác `@trace.bdd_version` (scalar) của `.feature`. T3b so từng entry với `.feature` tương ứng:

| Điều kiện | Severity | Auto-fix? |
|---|---|---|
| `.feature` **mới hơn** map | **Major** | ❌ cần người review lại §4/§4.5 rồi bump `@trace.revision` |
| Platform có `.feature` nhưng **vắng** trong map | Major | ✅ thêm entry sau khi xác nhận §4 đã phủ |
| Map có platform mà không còn `.feature` | Minor | ✅ xoá entry |

> **Major chứ không Minor** vì `/generate-code` DS3 thấy doc `approved` + 0 blocker-GAP sẽ lấy shape DTO/endpoint/error ở §4 **nguyên văn**. Contract dựng từ BDD cũ lan **thẳng** vào code — tệ hơn drift-về-code vì nó sai từ nguồn.
>
> Khi áp fix, **cấm chỉ sửa số trong map** cho ca "`.feature` mới hơn": làm thế là dán nhãn "đã đồng bộ" lên một contract chưa ai review. Platform còn finding `open` thì **giữ số cũ** để cờ `TECHDOC_STALE_VS_BDD` của [`/validate-traces`](09-validate-traces.md) còn sáng.

---

## HITL / Gate

- 🟠 SA duyệt tech-design; **read-only** review — chỉ báo findings, không tự sửa.
- 🔒 **T7 sign-off**: contract liên team chưa ký → không mở khoá code phía tiêu thụ.
- 🟡 **T3b (chặn mềm)**: còn finding T3b Major `open` → CHECKPOINT `Y/N` trước khi đặt `approved`. Mềm chứ không cứng như GATE §12 blocker-GAP, vì BDD hay bump vì lý do **không chạm contract** (sửa từ ngữ step) — người review là người biết.
- `@trace.status: approved` trên tech-design → `/generate-code` dùng §4 làm nguồn contract; `draft/in-review` hoặc còn blocker-GAP → chỉ WARN (không chặn). Tech-doc **cũ hơn BDD** cũng chỉ WARN — nhưng là WARN quan trọng nhất, vì nó xuất hiện đúng lúc shape §4 được lấy nguyên văn.

---

## Anti-pattern

- ❌ Tự "chế" shape DTO/endpoint khi API đã tồn tại — phải reverse-document as-is.
- ❌ Bỏ cổng ký T7 rồi để hai team hiểu contract khác nhau → rework tốn kém.
- ❌ Sinh code khi tech-design còn `draft` với contract chưa chốt.
- ❌ Bỏ qua `/map-testids` rồi để `/generate-code` **tự bịa** test-id — QC không có hợp đồng để bám, phải dò DOM.
- ❌ Ghi đè §4.5.6 khi giá trị lệch bản cũ — làm vỡ test-script đang chạy mà không ai biết.

---

## Bước tiếp theo (Next step)

Tech-design `approved` (+ ký T7 + §4.5.6 đã chốt) → **rẽ hai nhánh chạy song song**:

➡️ [Bước 6 · Code — `/generate-code`](06-code.md) — FE gắn `@trace.testid_attr` lên element theo §4.5.6
➡️ [Bước 8 · QC Automation — `/qc-design-test`](08-qc-automation.md) — QC dựng test case + script theo **cùng** §4.5.6

Hai nhánh **không dẫm chân nhau** vì cả hai đọc một hợp đồng đã đóng băng, không bên nào tự đặt test-id.
