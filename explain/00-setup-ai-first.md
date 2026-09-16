[← Explain Home](README.md) · [Next: /generate-architecture →](00b-generate-architecture.md)

# 00 · `/setup-ai-first` — Khởi tạo framework trong dự án

> **Một câu.** Dựng khung thư mục + config + `CLAUDE.md` + domain-knowledge cho một dự án, thích ứng theo **3 loại dự án** (single / umbrella / PO-spec).

---

## Vấn đề giải quyết

Mọi command sau đều đọc `project-context.yaml` + `CLAUDE.md` + `domain-knowledge/` để biết **đọc/ghi ở đâu, viết theo convention nào**. Không có bước setup → mọi lệnh halt ở gate "required context missing". Đây là bước dựng nền một-lần.

---

## Vị trí & tiền đề

- **Vị trí:** tiền pipeline (trước cả Discovery).
- **Đặc biệt:** lệnh này **bỏ qua Gate Step 1/2/3** (chưa có file input, chưa có project context) — chỉ chạy Step 0-B (model check). Project root = thư mục hiện tại.
- **Tiền đề:** một repo git (trống hoặc có code) + Node.js.

---

## Input / Output

**Input:** trả lời Q&A (loại dự án, service/stack, spec submodule path…).

**Output** (tuỳ `project_type`):

| project_type | Tạo gì |
|--------------|--------|
| **1 · Single** | `specs/{product-definition,domain-knowledge}/`, `.trace/`, `.agent/review/`, `CLAUDE.md` (đủ §1–§7), `project-context.yaml`, `business-dictionary.md`, `core-entities.md` |
| **2 · Umbrella** | Chỉ `.trace/` + `.agent/review/` ở umbrella root; `project-context.yaml` mode umbrella với `services` + `spec_source`; **không** tạo `CLAUDE.md` root (không có một stack đơn) |
| **3 · PO Spec repo** | `specs/{product-definition,domain-knowledge}/`, `feedback/`, `.agent/review/`, `CLAUDE.md` tối thiểu (§1+§7) |

---

## Các bước xử lý (chi tiết)

1. **Precondition check** — nếu đã có `CLAUDE.md` + `project-context.yaml` → hỏi regenerate? (an toàn chạy lại; mỗi bước đề nghị merge/skip).
2. **Step 0.5 · Loại dự án** — hỏi single / umbrella / PO-spec → lưu `project_type`.
   - Umbrella hỏi thêm: path spec submodule; một domain trải nhiều platform không? → **FORM A** (`domain:module`) hay **FORM B** (`domain:platform:module`, platform ∈ system/web/app).
3. **Step 1 · Cấu trúc thư mục** — tạo base dir theo `project_type` (bảng trên). Folder feature-package `specs/{domain}/{prd-slug}/` **không** tạo trước — lệnh generate tự tạo on demand.
4. **Step 2 · CLAUDE.md** — tạo từ template 7 section (§1 Overview, §2 Architecture, §3 Coding Standards, §4 Traceability, §5 Error Handling, §6 Build&Test, §7 Git). Umbrella bỏ qua; PO-spec chỉ §1+§7.
5. **Step 3 · project-context.yaml** — copy từ `templates/project-context.yaml`, điền `{{PLACEHOLDER}}`. Umbrella: verify/sửa `services` (path phải khớp tên submodule thật — generator để placeholder `TODO-…`).
6. **Step 4 · business-dictionary.md** — Canonical Terms / Banned Terms / Enum Registry. *(Umbrella skip — sống trong spec submodule.)*
7. **Step 5 · core-entities.md** — entity catalog máy-đọc-được (field, invariant, relationship). *(Umbrella skip.)*
8. **Step 6 · VS Code extension** — khuyến nghị cài "SDD Board" (Review Board + Living Docs UI).
9. **Step 7 · Verify** — checklist theo `project_type` (file/folder tồn tại; umbrella check submodule init).

---

## Checkpoint & Gate

- Chỉ có model check (Step 0-B) + xác nhận regenerate nếu đã setup.
- **Không** gate chặn — nhưng nhắc mạnh: sau setup phải **điền tay** `CLAUDE.md` + `project-context.yaml` + `domain-knowledge/` (AI *follow*, trống thì *đoán*).

---

## Cơ chế đặc biệt

- **3 project types** rẽ nhánh sâu — đặc biệt umbrella FORM A vs FORM B định hình cách `services` được khai báo và routing về sau.
- **On-demand feature folders** — không scaffold trước `specs/{domain}/{prd-slug}/`, giảm rác.
- **Umbrella tách spec/code** — spec sống ở `spec_source`; service submodule chỉ code + `.trace/`.

---

## 👓 Góc nhìn tối ưu

- **Điểm ma sát #1: điền tay sau setup.** `CLAUDE.md` + `core-entities.md` để trống là nguyên nhân gốc của "AI sinh sai convention/field". Có thể tối ưu bằng scaffold thông minh hơn (đọc `pom.xml`/`package.json` suy §1, §6) — hiện `/sync` đã làm phần này cho umbrella, single-service thì chưa.
- **Umbrella `services.path` placeholder `TODO-…`** dễ bị quên sửa → routing hỏng. Đáng thêm một validate cứng.
- **Trùng lặp checklist verify theo 3 project_type** — nếu thêm project_type mới phải sửa nhiều chỗ.

---

## Kết nối

**Trước:** — (đầu tiên) · **Sau:** [`/define-product`](01-define-product.md) để bắt đầu feature đầu tiên.
