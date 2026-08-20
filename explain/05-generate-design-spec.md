[← /review-context](04-review-context.md) · [Explain Home](README.md) · [Next: /generate-bdd →](06-generate-bdd.md)

# 05 · `/generate-design-spec` — Đặc tả thiết kế UI (chỉ FE/App)

> **Một câu.** Sinh đặc tả visual **bám Figma frame thật** (fetch qua Figma MCP) cho từng màn hình — chỉ cho service FE/App — làm nguồn cho BDD giao diện.

---

## Vấn đề giải quyết

BDD UI cần biết màn hình có gì, trạng thái nào, tương tác ra sao. Để `/generate-bdd` tự đoán → coverage thiếu/lệch Figma. Design-Spec chốt visual thật, 2 tầng ngôn ngữ, lái coverage UI.

---

## Vị trí & tiền đề

- **Vị trí:** Phase Design (nhánh FE/App), sau PRD approved.
- **Chặn cứng:** `platform_type = backend` → **STOP** (BE không có UI).

---

## Input / Output

**Input:** PRD approved (§4 User Flow/Wireframe) + **link Figma frame node-level** cho từng màn + `figma-components/{module}.md`.

**Output:** `{specs_dir}/{domain}/{prd-slug}/design-spec/{TICKET-ID}-design-spec-{active_platform}-{slug}.md` (platform: web / app / app-ios / app-android).

---

## Các bước xử lý (chi tiết)

1. **Platform Check** — suy `active_platform` từ module; backend → STOP; unknown → hỏi.
2. **Version Check (PRD drift)** — nếu design-spec đã có, so `Built from PRD` với Version PRD hiện tại:
   - Bằng → hỏi sinh lại? · Khác → CHECKPOINT drift (Y=cập nhật phần ảnh hưởng · F=sinh lại toàn bộ · N=huỷ).
   - Khi sinh/sinh lại: cập nhật `Built from PRD`, bump Version, **reset `Status: draft`** (design đổi → sign-off lại).
3. **Screen Discovery** — trích màn hình/page/modal từ PRD §4, trình PO xác nhận → `screen_list`.
4. **Figma Frame Links (bắt buộc)** — thu **một link node-level cho mỗi màn**:
   - Validate URL phải chứa `?node-id=...` (link file trần bị từ chối — AI không đọc được).
   - **Fetch qua Figma MCP** (`get_design_context`, `get_screenshot`) đọc layout/component/token thật; **không bịa** layout frame không thể hiện.
   - `none` → màn ❌ Missing.
5. **Gate mềm** — màn thiếu link → spec sinh dạng **draft** với cờ `missing_frames`, giữ `draft` tới khi đủ link đọc được; thêm AI Assumption cho mỗi màn thiếu.
6. **CHECKPOINT** → chờ Y.
7. **Generate** theo template: §1 Screen Inventory · §2 Screen Specs (mỗi màn: layout, component, state default/empty/loading/error) · §3 Interaction Patterns (responsive/hover/keyboard web; gesture/navigation app) · §4 Platform Considerations (a11y web; device/OS app) · §5 AC-UI (tiêu chí nghiệm thu design) · Appendix (tóm tắt Figma, design token đã tham chiếu).

---

## Checkpoint & Gate

- 🛑 Screen Discovery + CHECKPOINT trước generate.
- 🟠 Gate mềm: thiếu Figma frame → draft, không sign-off; `/generate-bdd` cảnh báo mềm nếu design-spec chưa approved.

---

## Cơ chế đặc biệt

- **Phụ thuộc Figma MCP** — đây là command duy nhất trong pipeline **fetch dữ liệu ngoài** (Figma). Chất lượng spec phụ thuộc trực tiếp link node-level + quyền truy cập.
- **2 tầng ngôn ngữ** — mô tả nghiệp vụ (PO) + tham chiếu component (Dev/Designer).
- **Drift theo PRD version** — `Built from PRD` giúp phát hiện PRD đổi sau khi design.

---

## 👓 Góc nhìn tối ưu

- **Điểm phụ thuộc ngoài mạnh nhất** — nếu Figma MCP không cấu hình/không quyền, mọi màn thành ❌ Missing → spec chỉ text. Đáng có fallback rõ ràng hơn (vd import screenshot thủ công).
- **Thu link per-màn thủ công** tốn thao tác PO với feature nhiều màn. Có thể tối ưu bằng đọc cả file rồi map node — nhưng bị chặn bởi giới hạn "AI không đọc link file trần".
- **Reset draft khi drift** đúng nguyên tắc nhưng tạo vòng sign-off lại; cân nhắc drift một phần (chỉ màn đổi).
- **Nhánh platform (web/app/app-ios/app-android)** nhân đôi một số section — chi phí bảo trì.

---

## Kết nối

**Trước:** [`/review-context`](04-review-context.md) (PRD approved) · **Sau:** Designer + PO sign-off → [`/generate-bdd`](06-generate-bdd.md).
