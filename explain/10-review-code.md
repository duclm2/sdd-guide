[← /generate-code](09-generate-code.md) · [Explain Home](README.md) · [Next: /dev-gen-test →](12-dev-gen-test.md)

# 10 · `/review-code` — Review code (read-only, 4 dimension)

> **Một câu.** Soát code vừa sinh qua **4 dimension** (traceability, layer, coding standards, spec compliance) — **chỉ báo findings, không tự sửa** — và có thể đề xuất ghi lesson.

---

## Vấn đề giải quyết

"AI tự fix tự review" = lặp lỗi. `/review-code` tách vai review khỏi generate: một vòng đọc độc lập đối chiếu code với spec + CLAUDE.md, báo cáo để Dev/Lead quyết.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Implementation, sau `/generate-code` (lần đầu).
- **Đặc biệt:** **read-only** — bỏ qua checkpoint ghi-file, không commit, không auto-fix.

---

## Input / Output

**Input:** code của UC + `.feature` + CLAUDE.md §2/§3 + trace `.tsv`.

**Output:** báo cáo findings (`Output Artifacts: none (read-only)`) + tuỳ chọn đề xuất lesson.

---

## Các bước xử lý (chi tiết)

Chạy checklist **5 dimension**:

| # | Dimension | Kiểm gì |
|---|-----------|---------|
| 1 | **Traceability** | 9 mục — xem bảng riêng dưới |
| 2 | **Layer Architecture** (CLAUDE.md §2) | Class đúng layer? Phụ thuộc đúng chiều? Không bypass layer? |
| 3 | **Coding Standards** (CLAUDE.md §3) | Naming? Response wrapper nhất quán? Exception không bị nuốt? Không magic number / log dữ liệu nhạy cảm? Transaction đúng? |
| 4 | **Spec Compliance** | Mỗi scenario có implementation? Không endpoint không tài liệu (code không có spec backing)? |
| 5 | **Seam & Stub** | Mồ côi khi ghép luồng — xem bảng riêng dưới |

### Dimension 1 — vì sao không chỉ là "có `@trace.implements` chưa"

`/generate-code` ghi **5 tag** trên mỗi entry-point. 4 tag ngoài `implements` là nguồn của drift detection, và thiếu chúng thì `/validate-traces` **mù ở file đó — im lặng**:

| Tag thiếu | `/validate-traces` mù cái gì |
|---|---|
| `@trace.prd_version` | Step 4 — `PRD_DRIFT` |
| `@trace.bdd_version` | Step 5c — `BDD_DRIFT` |
| `@trace.tech_doc_revision` | Step 5 — `TECHDOC_DRIFT` *(bỏ được nếu UC không có tech-doc)* |
| `@trace.source` | mất con trỏ ngược về spec |

→ thiếu bất kỳ tag nào = **major**, không phải minor.

Cộng thêm 3 mục cấu trúc:
- `@trace.source` trỏ file `.feature` **có thật**, đúng platform (`bdd/{platform}/…`) → sai path = major
- **File đa-UC**: mỗi UC một block 5 tag riêng trên method của nó. Gộp header, hoặc trỏ `@trace.source` vào **thư mục** = major *(3 tag version là scalar theo từng UC; và các lệnh tra tag bằng khớp chuỗi chính xác nên trỏ folder ra 0 kết quả)*
- **Tag mồ côi** — `@trace.implements`/`@trace.verifies` trỏ SC **không tồn tại** trong `.feature` = **critical** (`TRACE_ORPHAN`)

### Dimension 5 — lớp lỗi mà build xanh không thấy

`/generate-code` vừa sinh sổ `_seams.tsv` ở bước trước. Đây là chỗ **luồng ghép chạy vào no-op** hoặc **hàm thật không ai gọi** — build xanh, test từng-UC xanh, vẫn hỏng:

- sổ `_seams.tsv` còn dòng `status = READY` (= cờ 🔴) → critical
- class `*Stub*` còn là binding đang dùng trong khi hàng thật đã tồn tại → `SEAM_UNWIRED`, critical
- method còn `@trace.stub` rỗng trong khi `@trace.stub_owner` **đã gen** → `STUB_UNRESOLVED`, critical
- **method thật mồ côi** — logic thật bị đẻ **song song** thay vì lấp vào stub cũ (Fill-before-create trượt) → critical
- stub/seam **mới** thiếu tag hoặc thiếu dòng `PENDING` trong sổ → major *(nợ không ghi sổ = nợ tàng hình)*

> `SEAM_PENDING` / `STUB_PENDING` (owner UC chưa gen) là **bình thường** — chỉ nhắc.
>
> Trùng với `/validate-traces` Step 5b là **có chủ đích**: `/review-code` chạy sớm hơn (ngay sau codegen, trước khi có test), bắt sớm rẻ hơn.

Sau review → **Đề xuất ghi Lessons** (tuỳ chọn) qua step `capture-lesson`: nếu phát hiện lỗi lặp lại → đề xuất ghi guardrail vào `project-lessons.md` (L1 phân giải file → L2 dựng lesson → L3 dedup → L4 ghi → L5 xác nhận).

---

## Checkpoint & Gate

- Không gate chặn — báo cáo tư vấn. Dev tự sửa (không phải AI auto-fix).
- **Verdict:** bất kỳ finding critical ở dimension 5 → `NEEDS_FIX`, **kể cả khi build xanh và test từng-UC xanh** — đó chính là loại lỗi hai thứ đó không bắt được.

---

## Cơ chế đặc biệt

- **Read-only nghiêm ngặt** — thể hiện nguyên tắc "review tách khỏi generate".
- **Tự phát hiện trace stale** → nhắc chạy `/validate-traces` trước để review chính xác.
- **Cầu nối với `/learn`** — review là nơi tự nhiên phát hiện pattern lặp → đề xuất lesson.

---

## 👓 Góc nhìn tối ưu

- **Không auto-fix có chủ đích** — nhưng nghĩa là mọi finding cần vòng người sửa. Với lỗi cơ học (thiếu tag `@trace`), cân nhắc một `--fix` an toàn giống `/review-context`?
- **Checklist tĩnh 4 dimension** (không fan-out) — với file code lớn có thể sót. Nhẹ hơn review tài liệu vì scope hẹp (1 UC).
- **Phụ thuộc `.tsv` không stale** — nếu stale, dimension 1 kém tin cậy; đã có cơ chế nhắc validate-traces.
- **Đề xuất lesson là điểm mạnh** — biến review thành tích luỹ tri thức; nhưng phụ thuộc người xác nhận L5.

---

## Kết nối

**Trước:** [`/generate-code`](09-generate-code.md) · **Sau:** pass → [`/dev-smoke-test`](14-dev-smoke-test.md) hoặc tạo PR; lỗi lặp → [`/learn`](27-learn.md).
