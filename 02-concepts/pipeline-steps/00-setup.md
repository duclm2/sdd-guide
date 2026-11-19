[← Pipeline Steps](README.md) · [Next: Discovery →](01-discovery.md)

# Bước 0 · Setup — Dựng khung dự án (Project Bootstrap)

> **Tóm tắt.** Chạy **một lần** cho mỗi dự án để tạo cấu trúc thư mục, config và `CLAUDE.md` mà mọi workflow về sau dựa vào.
> **Command:** `/setup-ai-first` (hoặc `npx @educa-corp/sdd-framework --init`)

| | |
|---|---|
| **Giai đoạn** | Setup (tiền pipeline) |
| **Owner** | Admin / Tech Lead |
| **Tần suất** | Một lần khi khởi tạo dự án (single) hoặc khi thêm service (umbrella) |
| **HITL** | ⚪ Thấp — chỉ chọn mode & module |

---

## Mục đích (Purpose)

Framework cần biết **dự án của bạn nằm ở đâu và dùng stack gì** trước khi có thể sinh spec/code đúng chỗ, đúng convention. Bước Setup:

- Tạo bộ khung thư mục chuẩn (`specs/`, `.trace/`, `.agent/`, `.claude/`).
- Sinh **`project-context.yaml`** — single source of truth cho **paths, mode, services, conventions, tech_stack**.
- Tạo **`CLAUDE.md`** phân tầng (root + overlay theo service) — nơi tích luỹ kiến trúc & coding standards để AI *follow* thay vì *invent*.
- Cài các **module stack** đã chọn (java-spring, react, flutter, qc-playwright-ts, qc-wdio-appium…).

> Bỏ qua bước này = mọi lệnh sau không biết đọc/ghi ở đâu → halt ở gate "required context missing".

---

## Input (Đầu vào)

- Một thư mục dự án (repo git — có thể trống hoặc đã có code).
- Quyết định **loại dự án**: `single` (một repo) · `umbrella` (nhiều service submodule + một spec repo dùng chung) · **PO Spec repo** (chỉ docs — PRD + design-spec, không có code chạy được).
- Danh sách **tech stack** của các service (để cài đúng module).
- (Umbrella) đường dẫn tới **spec repo** dùng chung.

---

## Output (Đầu ra)

| Artifact | Vai trò |
|----------|---------|
| `.agent/` | Bản runtime của framework: `commands/`, `steps/`, `rules/`, `skills/`, `templates/`, `modules/`, `hooks/` |
| `.claude/` | Cấu hình Claude Code (slash command, settings) |
| `.agent/project-context.yaml` | **Config runtime** — mọi workflow đọc để biết paths/mode/conventions |
| `CLAUDE.md` (root) | Luật chung + trỏ tới overlay. Umbrella: thêm `{service}/CLAUDE.md` theo stack |
| `specs/` + `.trace/` | Khung thư mục cho PRD/BDD/tech-docs/design-spec và trace state |
| `specs/domain-knowledge/` | Nơi chứa business dictionary & tri thức miền |

---

## Ai làm gì (Roles & responsibilities)

| Ai | Việc |
|----|------|
| 👤 **Admin/Lead** | Chọn mode single/umbrella, khai báo services & stack, trả lời Q&A setup |
| 🤖 **AI** | Copy framework files, sinh `project-context.yaml` từ template (token substitution), cài module, dựng `CLAUDE.md` |
| 👤 **Lead (sau setup)** | **Điền nội dung** `CLAUDE.md` (§2 kiến trúc, §3 coding standards, §5 error handling) và `domain-knowledge/` — phần AI không tự biết |

---

## Câu hỏi cần trả lời (Questions this step answers)

- Dự án này là **single**, **umbrella** (nhiều service) hay **PO Spec repo** (chỉ docs)?
- Mỗi service dùng **stack** gì → cài module nào?
- Spec sẽ nằm ở đâu (`specs_dir`), trace ở đâu (`.trace/`)?
- Lệnh build/test của từng stack là gì (`conventions.build_command`)?

---

## Framework xử lý thế nào (Mechanics)

1. **Init** — `npx … --init` hoặc `/setup-ai-first` copy `.agent/*` và `.claude/*` vào dự án.
2. **Token substitution** — điền `{{PROJECT_NAME}}`, paths, services… vào `templates/project-context.yaml` → sinh `.agent/project-context.yaml`.
3. **Cài module** — mỗi module mang `module.yaml` + `stack-profile.yaml` (module dev khai `build`, `architecture`, `coding_standards`, `testing`, `trace_tags`; module QC khai thêm `layout`, `locator_priority`, `reporting`). Skill **đọc profile** thay vì hardcode stack.
4. **Dựng CLAUDE.md** — tạo file root; umbrella thì thêm overlay cho từng service. Kèm `domain-knowledge/business-dictionary.md` + `core-entities.md` (khung để Lead điền).
5. **Cổng chặn bằng máy** *(khuyến nghị mạnh, bỏ qua với PO Spec repo)* — pre-push hook (sổ trace hỏng cấu trúc) và/hoặc CI workflow (chặn PR khi có cờ 🔴).
6. **Verify** — checklist thư mục/file theo loại dự án (`specs/`, `.trace/`, `project-context.yaml`, `CLAUDE.md`…).

> `project-context.yaml` dùng `{domain}` và `{prd-slug}` làm biến — mọi path đều theo bố cục **feature-package**: `specs/{domain}/{prd-slug}/…`. Xem [Configuration](../../04-reference/configuration.md).

---

## HITL / Gate

- Không có gate chặn — đây là bước dựng khung.
- Điểm cần con người: **chọn mode/module đúng** và **điền `CLAUDE.md` + `domain-knowledge/`** sau khi setup. Đây là "trí nhớ nền" mà toàn bộ pipeline sẽ dựa vào.

---

## Anti-pattern

- ❌ Bỏ trống `CLAUDE.md` rồi than "AI sinh code sai convention" — AI *follow* file này; trống thì nó *đoán*.
- ❌ Sửa thẳng `.agent/commands/*.md` — sẽ mất khi `/update-framework`. Nội dung của bạn nằm ở `CLAUDE.md`, `project-context.yaml`, `domain-knowledge/` (không bị ghi đè).
- ❌ Chọn `single` cho một hệ nhiều service — sau này khó tách spec/trace dùng chung.

---

## Bước tiếp theo (Next step)

Điền xong `CLAUDE.md` + `domain-knowledge/` → bắt đầu feature đầu tiên:

➡️ [Bước 1 · Discovery — `/define-product`](01-discovery.md)
