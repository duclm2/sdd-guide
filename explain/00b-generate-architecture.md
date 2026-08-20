[← Explain Home](README.md) · [Prev: /setup-ai-first](00-setup-ai-first.md) · [Next: /define-product →](01-define-product.md)

# 00b · `/generate-architecture` — Sinh / làm mới Architecture Context (SSOT)

> **Một câu.** Thu thập kiến trúc từ **mọi nguồn sẵn có** (config + tài liệu + code) rồi **phỏng vấn lấp chỗ trống** để dựng `architecture.md` — nguồn chân lý kiến trúc cắt ngang, do SA/Tech Lead verify.

---

## Vấn đề giải quyết

`/generate-tech-docs` cần biết kiến trúc thật của hệ thống (tầng, ranh giới truy cập dữ liệu, auth, multi-tenant…) để tech-design không "tự chế" mỗi feature một kiểu. Nguồn đó là `architecture.md`.

Nhưng một template kiến trúc đầy đủ có ~140 ô. Bắt SA điền tay từ giấy trắng là **không tối ưu** — vừa nản, vừa trùng lặp với thứ đã nằm trong config/tài liệu/code. Lệnh này đảo ngược gánh nặng: **máy hút cái đã biết, người chỉ trả lời cái chưa biết**.

---

## Vị trí & tiền đề

- **Vị trí:** bước nền (foundation), ngoài vòng lặp per-feature. Chạy sau `/setup-ai-first`, một lần rồi refresh khi kiến trúc đổi.
- **Đặc biệt:** bỏ qua Gate Bước 1 (không có feature-file input). Vẫn chạy model check + context-loader.
- **Tiền đề:** đã có `project-context.yaml` + `CLAUDE.md`. Brownfield thì cần code để scan; greenfield thì dựa vào phỏng vấn.

---

## Input / Output

**Input:** `$ARGUMENTS` tuỳ chọn — `{service-path}` (umbrella), `--from=<paths>` (tài liệu có sẵn), `--section=<slug>` (điền dần 1 mục), `--interview` (ép phỏng vấn).

**Output:** `specs/architecture.md` (hoặc `{service}/specs/architecture.md`) — theo **tier** (core / conditional / ops), mục chưa làm để dạng **STUB**, frontmatter `verified_by: AI-draft` chờ người verify.

---

## Các bước xử lý ⭐

| Bước | Làm gì |
|------|--------|
| **1 · Phân giải** | Target file + chế độ (brownfield/greenfield) + parse cờ. File đã `verified_by: {người}` → sang refresh có kiểm soát (không đè). |
| **2 · Thu thập đa nguồn** | Theo ưu tiên: **config** (`project-context`+`CLAUDE.md`) → **tài liệu** (`--from`) → **scan code**. Mỗi field ghi kèm nguồn + độ chắc chắn; đánh dấu field còn trống / xung đột. |
| **3 · Phỏng vấn thích ứng** | Báo cáo coverage → **Pha 1 sàng lọc** (thẻ dễ hiểu, bỏ câu đã có đáp án) → **Pha 2 đào sâu** (drill loop có ràng buộc, chỉ trên mục đã "CÓ", lối thoát "để sau"). |
| **4 · Lắp ráp theo tier** | core luôn viết · conditional viết nếu bật, else STUB · ops mặc định STUB. Chèn `<!-- nguồn -->` / `<!-- ⚠️ xung đột -->`. Frontmatter trust-gate. |
| **5 · `--section`** | Chỉ lấp/refresh 1 mục (điền dần), gỡ cờ stub. |
| **6 · Refresh** | File đã verify → chỉ xuất drift, không đè. |
| **7 · Bàn giao** | Nhắc SA verify + đổi `verified_by`. |

---

## Checkpoint & Gate

- 🛑 Nếu file đang `AI-draft` → hỏi trước khi regenerate đè.
- 🔒 **Trust-gate `verified_by`**: chỉ khi là **người thật**, `/generate-tech-docs` mới coi doc là ràng buộc chính thức; `AI-draft` bị gắn ⚠️ và thua CLAUDE.md/BDD khi mâu thuẫn.
- Umbrella không có service path → DỪNG (kiến trúc là per-service).

---

## Cơ chế đặc biệt

- **Điền dần (progressive):** mục chưa làm là STUB, lấp sau bằng `--section=<slug>` — không phải regenerate cả file.
- **Skip-if-answered:** câu phỏng vấn nào (kể cả follow-up) đã có đáp án từ config/tài liệu/code thì **không hỏi**.
- **Phỏng vấn 2 pha có ràng buộc:** chỉ đào sâu trong phạm vi SA đã trả lời CÓ, gộp 2–3 câu/lượt, luôn có "để sau" → loop chắc chắn kết thúc.
- **Tier lọc context:** `/generate-tech-docs` chỉ nạp **core + conditional**, bỏ **ops** → giảm nhiễu.

---

## 👓 Góc nhìn tối ưu

- **Chi phí SA:** từ "điền 140 ô" → "trả lời vài thẻ + duyệt draft". Điểm mấu chốt là chất lượng bước **hút nguồn** (Bước 2) — hút càng tốt, hỏi càng ít.
- **Rủi ro:** draft `AI-draft` có thể sai; trust-gate ép người verify trước khi nó ảnh hưởng code-gen. Đừng để doc kẹt mãi ở `AI-draft`.
- **Phụ thuộc:** chất lượng phụ thuộc `--from` (tài liệu tốt) + code rõ ràng. Greenfield dựa nhiều vào phỏng vấn.

---

## Kết nối

**Trước:** [`/setup-ai-first`](00-setup-ai-first.md) (đã có config).
**Sau:** [`/generate-tech-docs`](07-generate-tech-docs.md) nạp `architecture.md` (Bước 0.5 [ARCH]) để tech-design bám kiến trúc.
**Người sở hữu:** SA / Tech Lead — xem [Guide · Architect / SA](../03-guides/architect.md).
